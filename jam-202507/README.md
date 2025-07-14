# Skye's July 2025 Jam

This is my entry for Skye's Jam of July 2025.

The schematic builds and runs a base around the core until a mega is produced. Unless the resources are scarce or far away, the mega is produced within 30 minutes.

Prerequisites:

* A single Shard core must be present on the map.
* 13x13 area centered on the core must be empty (allow buildings to be built).
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
* [Core Builder](CoreBuilder.mnd) (the leftmost logic processor): the main driver of the logic. Responsible for going through all the intermediate steps of the process. Controls the original poly most of the time.
* [Display](Display.mnd) (middle column, top): responsible for displaying information on the display and the bottom message.
* [Ore Locator](OreLocator.mnd) (middle column, middle): contains the logic for searching for the ore and part of the logic for laying out blocks not included in the core base (drill infrastructure).
* [Matrices Generator](Matrices.mnd) (middle column, bottom): logic for scanning terrain for ore and build obstacles. Extracted from Ore Locator due to a large code size; however, this also allows two terrain scanning operations to run in parallel, speeding up the thorium drill building operation significantly. 
* Microprocessors (top to bottom):
  * Timer: measures time by counting instruction executions. Allows measuring time when the Time Control mod is used. Consists of a single `op add time time 1` instruction.
  * [Flare Controller](FlareController.mnd): controls the flares. Flares are used to fetch titanium from the drills (each flare serves one titanium drill, for simplicity).
  * [Mono Controller](MonoController.mnd): controls the monos. Monos mine the most needed resource (copper, lead or sand) depending on the core stockpile and the next planned building.  
  * [Poly Controller](PolyController.mnd): controls the polys (except the original one). Polys mine coal exclusively.
  * [Block Builder](BlockBuilder.mnd): contains logic for building blocks at given positions. Extracted from Core Builder due to a large code size.
  * [Core Controller](CoreController.mnd): controls blocks in the core base, managing resource production. (The actual processor doing this is built in the base and uses this processor only for copying the logic from it).

The top-level process is mostly uninteresting. The Core Builder goes through a predefined sequence of goals (i.e., building a block or a unit). Each goal comes with a resource requirement, and the Core Builder ensures the requirement is met before building the block or unit. No sophistication is used here: the resources are gathered using fixed routines (e.g., mine ore, bring ore to a block, bring ore to the core, fetch ore from a drill, etc.), and each routine runs into completion. This also means the routine cannot end prematurely: a mining operation is not interrupted when the requirements are met. This is probably efficient, as the ores mined will be needed later anyway, and terminating the mining prematurely would actually waste time, but it is visually unappealing.

Although a lot of details had to be ironed out for this, some specific tasks might be more interesting and are described in greater detail below.

### Searching for ores

When searching for ores, a unit is flown in an outward-spiraling pattern, locating the ore of interest using `ulocate`. Locations close enough to the unit are inspected and evaluated (the unit flies there and reads the terrain). Visited locations are stored in a memory bank to avoid processing the same location repeatedly, which would be quite time-consuming.

The map is searched until the farthest distance is reached, or until a good enough spot is found. The criterion for a good enough spot is lowered as the drone gets farther from the core, to shorten up the search.

The ore search is used on three different occasions:

* building the fist titanium drill,
* building additional titanium drills,
* building the thorium drill and a related infrastructure.

### Optimal drill placement

`ulocate` provides a single tile with the ore. The best position for a drill at a given location is determined, for a 2x2, 3x3, or 4x4 drill. The 4x4 version is used for the second titanium search; up to four pneumatic drills are placed on the 4x4 location found. This reuses existing code for a single drill placement and avoids potentially complicated logic for placing separate drills close to each other optimally.

The 2x2 and 3x3 drills are placed optimally within a 5x5 area. The 4x4 drill is placed optimally within a 7x7 area. The optimal location is determined by counting the number of tiles under the drill, as well as detecting possible obstacles that would prevent the drill from being built. These constraints result in the following number of possible drill placements within the respective areas:
 
* 2x2 drill: 16 different placements,
* 3x3 drill: 9 different placements,
* 4x4 drill: 16 different placements.

For each possible placement, a score is calculated; the best score determines the optimal placement. Score is calculated by summing up individual tile values: 0 for an empty tile, 1 for a tile with an ore, and 32 for a solid tile (actually, these values are divided by 255 for reasons stated below). Score greater or equal to 32 indicates an invalid position, while other values give the number of tiles under the drill.

The actual trick here is making the calculation fast. To this end, all tiles in a row within the area are scanned, and a partial score is calculated for the (three or four) possible horizontal drill positions. These values are then packed together into a single value using `packcolor` (for the 4x4 drill, the values are capped at 32, but for the other two dimensions this is not even necessary) - this is why the original tile scores are divided by 255, as required by `packcolor`.

The final score for each possible vertical drill position is calculated by summing up the partial row scores for each row within the position and then unpacking the values into four column scores using `unpackcolor`. This greatly reduces the total number of additions needed, and using `packcolor` and `unpackcolor` saves the bit manipulation operations.   

## Known limitations

TBD.
