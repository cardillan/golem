# Base Builder

This schematic was originally created as an entry for Skye's Jam of July 2025.

The schematic builds and runs a base around the core until a mega is produced. Unless the resources are scarce or far away, the mega is produced within 30 minutes.

Prerequisites:

* A single Shard core must be present on the map.
* 13x13 area centered on the core must be empty (allow buildings to be built).
* The terrain must not block the multiplicative reconstructor's exit. 
* Copper, lead, coal, titanium and thorium ore, and a sand tile, must be present on the map. The titanium and thorium ore must allow a drill to be built on them, and the area around thorium must allow building required infrastructure.
* If the Time Control mod is used, speeds higher than 8x may lead to failures (the processors do not keep up, at speeds greater than 16x it seems that even mining gets broken).

The world processor in the schematic sets up a single poly near the core.

The switch changes the information shown on the display.

The game measures and displays the time it takes to build blocks and units. The time measurement is compatible with the Time Control mod, for 1x, 2x, and 4x speeds. For other speeds the time measurement is imprecise. 

A more detailed description of the logic used will be added later.

## Basic overview

The mega is produced by building a base first. Flares, monos, and polys are also produced to support the base and (hopefully) speed up the entire progress. The base is built in a pre-determined layout, and almost all operations are carried in a fixed order. The only variability is in the number of drills built to extract thorium. The program ends when the first mega is produced.

Brief description of the processors:

* The world processor for spawning the poly (allows the schematic to be put on any map and run)
* [Base Builder](BaseBuilder.mnd) (top-left logic processor): the main driver of the logic. Responsible for going through all the intermediate steps of the process.
* [Builder Controller](BuilderController.mnd) (top-right processor): Controls the original poly (the builder), uses it to fullfill individual goals prescribed by the base builder.
* [Ore Locator](OreLocator.mnd) (middle-left logic processor): contains the logic for searching for the ore and part of the logic for laying out blocks not included in the core base (drill infrastructure).
* [Matrices Generator](Matrices.mnd) (middle-right logic processor): logic for scanning terrain for ore and build obstacles. Extracted from Ore Locator due to a large code size; however, this also allows two terrain scanning operations to run in parallel, speeding up the thorium drill building operation significantly. 
* [Display](Display.mnd) (bottom-left logic processor): responsible for displaying information on the display and the bottom message.
* Microprocessors (top to bottom, left to right):
  * [Mono Controller](MonoController.mnd): controls the monos. Monos mine the most needed resource (copper, lead or sand/scrap) depending on the core stockpile and the next planned building.  
  * [Flare Controller](FlareController.mnd): controls the flares. Flares are used to fetch titanium from the drills (each flare serves one titanium drill, for simplicity).
  * [Resource measurement](Measurement.mnd): measures the resources stored in the core relative to the target and stores the information in the right-hand memory bank.
  * [Poly Controller](PolyController.mnd): controls the polys (except the original one). Polys mine coal exclusively.
  * Timer: measures time by counting instruction executions. Allows measuring time when the Time Control mod is used. Consists of a single `op add time time 1` instruction.
  * [Core Controller](CoreController.mnd): controls blocks in the core base, managing resource production. (The actual processor doing this is built in the base and uses this processor only for copying the logic from it).

The top-level process is mostly uninteresting. The Base Builder goes through a predefined sequence of goals (i.e., building a block or a unit). Each goal comes with a resource requirement, and the Builder Controller ensures the requirement is met before building the block or unit. No sophistication is used here: the resources are gathered using fixed routines (e.g., mine ore, bring ore to a block, bring ore to the core, fetch ore from a drill, etc.), and each routine runs into completion.

Although a lot of details had to be ironed out for this, some specific tasks might be more interesting and are described in greater detail below.

### Searching for ores

When searching for ores, a unit is flown in an outward-spiraling pattern, locating the ore of interest using `ulocate`. Locations close enough to the unit are inspected and evaluated (the unit flies there and reads the terrain). Visited locations are stored in a memory bank to avoid processing the same location repeatedly, which would be quite time-consuming.

The map is searched until the farthest distance is reached, or until a good enough spot is found. The criterion for a good enough spot is lowered as the drone gets farther from the core, to shorten up the search.

The ore search is used on three different occasions:

* building the first titanium drill,
* building additional titanium drills,
* building the thorium drill and a related infrastructure.

The thorium ore search almost immediately follows the second search for the titanium ore. Arguably, these two search phases could be combined, and some time might be saved on maps where the ore is located far away, but this would increase the code size considerably.  

### Optimal drill placement

The described functionality is implemented [in the `findBestPosition()` function in Matrices.mnd](https://github.com/cardillan/golem/blob/5e0cb22fb89be1cc94957b2db68dbe002690ca84/jam-202507/Matrices.mnd#L135).

`ulocate` provides a single tile with the ore. The best position for a drill at a given location is determined, for a 2x2, 3x3, or 4x4 drill. The 4x4 version is used for the second titanium search; up to four pneumatic drills are placed on the 4x4 location found. This reuses existing code for a single drill placement and avoids potentially complicated logic for placing separate drills close to each other optimally.

The 2x2 and 3x3 drills are placed optimally within a 5x5 area. The 4x4 drill is placed optimally within a 7x7 area. The optimal location is determined by counting the number of tiles under the drill, as well as detecting possible obstacles that would prevent the drill from being built. These constraints result in the following number of possible drill placements within the respective areas:
 
* 2x2 drill: 16 different placements,
* 3x3 drill: 9 different placements,
* 4x4 drill: 16 different placements.

For each possible placement, a score is calculated; the best score determines the optimal placement. Score is calculated by summing up individual tile values: 0 for an empty tile, 1 for a tile with an ore, and 32 for a solid tile (for computation these values are divided by 255 for reasons stated below). Score greater or equal to 32 indicates an invalid position, while other values give the number of tiles under the drill.

The actual trick here is making the calculation fast. To this end, all tiles in a row within the area are scanned, and a partial score is calculated by summing up tile values of the (three or four) possible horizontal drill positions. These values are then packed together into a single value using `packcolor` (for the 4x4 drill, the values are capped at 32, but for the other two dimensions this is not even necessary) - this is why the original tile scores are divided by 255, as required by `packcolor`.

The final score for each possible vertical drill position is calculated by summing up the partial row scores for each row within the position and then unpacking the values into four column scores using `unpackcolor`. This greatly reduces the total number of additions needed, and using `packcolor` and `unpackcolor` avoids a lot of bit manipulation operations.

When the optimal drill position obtained by this process is different from the original ore location, a new area, centered around the optimal drill position, is evaluated again using the same process. This is repeated as long as the drill score increases. This process is capable of following a vein of ore to a better location in some circumstances.

### Dynamic block placement

The thorium drill needs additional infrastructure nearby. A water extractor touching the drill, as well as a container, is optional, while a steam generator with an adjacent water extractor, a battery, a solar panel, and a power node connecting it all up are required.

The placement of the drill itself is done as described above. Once the drill position is determined, it's fixed (this limitation precludes retrying drill placement in case the surrounding infrastructure runs into problems, which would be possible, but too computationally expensive).

The remaining blocks, unlike the drills, do not require a specific position for optimal operation but are subject to two constraints:

* they cannot be built on a solid block (the same as drills),
* they must not touch another already existing block (touching by corners is okay).

The second constraint is to prevent placing the water extractor next to a pneumatic drill built earlier. When this happens, the pneumatic drill might drain all water from the extractor, making the steam generator, and consequently the laser drill, non-operational.

The placement of these additional blocks is planned on a 7x7 grid centered on the drill. This area contains 49 tiles, for which two bitmasks are created: a bitmask of solid blocks, and a bitmask of tiles adjacent to existing blocks (these bitmasks are built in parallel, and each tile is scanned only one for each bitmask). The bitmasks are then combined for a single "terrain" bitmask containing ones at bits corresponding to blocked tiles, and zeroes at bits corresponding to free tiles.

* [Solid tile bitmask](https://github.com/cardillan/golem/blob/5e0cb22fb89be1cc94957b2db68dbe002690ca84/jam-202507/Matrices.mnd#L25)
* [Adjacent blocks bitmask](https://github.com/cardillan/golem/blob/5e0cb22fb89be1cc94957b2db68dbe002690ca84/jam-202507/OreLocator.mnd#L426)

If the area of interest is located close to the base, possible base areas are also blocked in the bitmask based on the distance from the base and base size. 

To find a position for a block, a bitmask for a block is created. To evaluate different block positions, [the block bitmask is shifted accordingly](https://github.com/cardillan/golem/blob/5e0cb22fb89be1cc94957b2db68dbe002690ca84/jam-202507/OreLocator.mnd#L322), and the resulting bitmask is compared (using binary and) to the bitmask of free tiles. The first free position found is selected.

To place blocks that have to touch the side of the drill, the [drill itself and the corners of the area bitmask are explicitly blocked](https://github.com/cardillan/golem/blob/5e0cb22fb89be1cc94957b2db68dbe002690ca84/jam-202507/OreLocator.mnd#L477). This ensures these 2x2 blocks cannot be placed diagonally from the drill. Positions of these blocks are marked in the terrain bitmask by or-ing them in. 

The water extractor, the steam generator are placed as a single 2x4 block, either horizontally or vertically, using the updated terrain bitmask, and then the three 1x1 blocks (battery, solar panel, and power node) are also added. The 1x1 blocks must not be placed in corners or on tiles adjacent to the corners, otherwise the distance between the power node and either of the two other blocks might be too large for a connection to be made.  

If it is not possible to place all these blocks within the original 7x7 area (this may actually easily happen), four more areas, each shifted by three tiles in both x and y directions, are evaluated. The terrain needs to be scanned anew for these areas, but the already planned blocks (the drill and the two touching 2x2 blocks) are [shifted accordingly](https://github.com/cardillan/golem/blob/5e0cb22fb89be1cc94957b2db68dbe002690ca84/jam-202507/OreLocator.mnd#L271) and combined with the new terrain mask. An additional power node is built when the original power node is not able to reach all powered blocks; connection is ultimately ensured via the drill.  

## Known limitations

* If the only thorium ore is too close to the base, the logic might fail to build the drill, although a solution actually exists.
* If the area around a thorium ore is very constrained, the logic might fail to find a solution for planning the drill infrastructure, even if one exists. In this case, no efforts for finding a new thorium location will be made.