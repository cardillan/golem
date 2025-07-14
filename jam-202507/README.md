# Skye's July of 2025 Jam

This is my entry for Skye's Jam of July 2025.

The schematic builds and runs a base around the core until a mega is produced. Unless the resources are scarce or far away, the mega is produced within 30 minutes.

Prerequisites:

* A single Shard core must be present on the map.
* 13x13 area centered on the core must be empty (allow buildings to be built).
* Copper, lead, coal, titanium and thorium ore, and a sand tile, must be present on the map. The titanium and thorium ore must allow a drill to be built on them, and the area around thorium must allow building required infrastructure.
* If the Time Control mod is used, speeds higher than 8x may lead to failures (the processors do not keep up, at speeds greater than 16x it seems that even mining gets broken).

The world processor in the schematic sets up a single poly near the core.

The switch changes the information shown on the display.

The game measures and displays the time it takes to build blocks and units. The time measurement is compatible with the Time Control mod, for 1x, 2x and 4x speeds. For other speeds the time measurement is imprecise. 

A more detailed description of the logic used will be added later.
