# Measurements

This directory contains a collection of schematics oriented at measuring and displaying various quantities

## Item Counter

![Screenshot of the schematics in action](item-counter.png)

The item counter (and a smaller, microprocessor-based variant) is capable of counting items being loaded onto a plastanium conveyor. The loading spots of the plastanium conveyors to be monitored must be linked to the processor. The processor detects new batches being formed and updates a running total of items detected.

The standard Item Counter can handle up to 16 conveyors at once, Item Counter Micro can handle up to 4 plastanium conveyors without missing a batch being created, under these conditions:

1. The game runs at 60 FPS or higher.
2. If the processor is boosted to the same level as the plastanium conveyors by overdrive projectors or domes, the conveyors can be fed at the maximum possible speed (i.e. by feeding them from several directions and/or using bridges to increase item supply).
3. If the processor isn't boosted, but the plastanium conveyors are (up to 250%), the item Counter is guaranteed to work as long as each plastanium conveyor is fed at a rate of 10 items/sec. This means the conveyors are fed from a single source (a factory, an unloader or a conveyor; when fed from a bridge, the bridges must not be chained to increase the flow rate).
4. The plastanium conveyors must not be fed directly from other plastanium conveyors (e.g. by placing loading and unloading spot next to each other). In these situations, the whole batch gets transferred instantaneously and the processor can't detect them.

The total amount of items is displayed in the message block.

The Item Counter micro comes with a built-in memory. The Item Counter needs a separate external memory to write the running total to. Other schematics in this directory are capable of computing and graphing the flow rate from these values.

Linking or unlinking blocks to/from the processor causes the processor to reinitialize and rescan linked blocks, potentially missing some items that were sent during the reinitialization. Pressing the switch reinitializes the processor and also resets the total.

Linked blocks which are unsupported are ignored, and they do not slow down the processor. When linking more than the supported number of plastanium conveyors, the processor gets stuck at reinitialization until the problem is corrected. The processor doesn't detect when a plastanium conveyor is linked incorrectly (that is by a segment different from their loading spot).

To count items in the middle of a plastanium belt, the flow must be interrupted by different kind of transport, such as bridges or titanium conveyors:

![Screenshot of the schematics in action](item-counter-micro.png)

## Item Rate Meter

Monitors item counts recorded by item counter and computes rate of item production/consumption in items/tick or items/second.

## External memory usage

> [!TIP]
> This chapter documents layouts of data in memory cells and memory banks used by the schematics in this directory.

Schematics in this directory store measurements in memory cells or memory banks. Measurement schematics write measured quantities into these memory banks, while graphing schematics read values to be graphed from them. The following setups are supported:

* Item Counter: schematic to count produced or distributed items.
* Item Rate Meter/Level Meter: schematics to record quantities over time:
  * Item Rate Meter computes and records item production/consumption rate from data provided by Item Counter.
  * Level Meter records levels of items/liquids stored in containers or energy stored in batteries.
* Level Display: displays the data recorded by Item Rate Meter/Level Meter as a graph.

Item Counter/Item Rate Meter may also be paired up with Factory Monitors to display current production rate on a factory status display.

All these schematics may use one or two memory blocks for data.

### Single memory block

A memory bank must be used as a single memory storage if graphing is required, as memory cell doesn't have enough capacity to hold all data. If graphing is not required and just a current value is being consumed by some component, a memory cell is adequate. Memory layout for a single memory block is this:

| Index           | Writes | Reads | Data                                                                                             |
|-----------------|:------:|:-----:|--------------------------------------------------------------------------------------------------|
| 0 .. 49         |   M    |   M   | Cache used by Item Rate Meter/Level Meter for smoothing out (averaging) raw data measurements    |
| 50 .. SIZE - 10 |   M    |  M,D  | Circular buffer holding quantities recorded over time (typically sampled at a frequency of 1 Hz) |
| SIZE - 9        |   M    |   D   | Foreground graph color                                                                           |
| SIZE - 8        |   M    |   D   | Background graph color                                                                           |
| SIZE - 7        |   M    |   D   | Start of the circular buffer (50)                                                                |
| SIZE - 6        |   M    |   D   | End of the circular buffer, exclusive (SIZE - 7)                                                 |
| SIZE - 5        |   M    |   D   | Head of the circular buffer (i.e. latest value written)                                          |
| SIZE - 4        |  M,D   |  M,D  | Current minimum of recorded quantities                                                           |
| SIZE - 3        |  M,D   |  M,D  | Current maximum of recorded quantities                                                           |
| SIZE - 2        |   M    |   D   | Current measurement (item rate or level, as measured by Item Rate Meter/Level Meter)             |
| SIZE - 1        |   C    |   M   | Total count of items as computed by Item Counter                                                 |

SIZE is the size of the memory bank (512) or memory cell (64).

_Writes_ and _Reads_ specifies which components write/read the given data:

* `C` denotes Item Counter
* `M` denotes Item Rate Meter/Level Meter
* `D` denotes Display.

### Two memory blocks

When two memory blocks are used, they're denoted as _primary_ and _secondary_. The primary memory block may be a memory cell or a memory bank. The secondary memory block should be a memory bank, as it holds the data to be displayed. When no graphing is required, using two memory blocks is superfluous.

**Primary memory**

| Index         | Writes | Reads | Data                                                                                             |
|---------------|:------:|:-----:|--------------------------------------------------------------------------------------------------|
| 0 .. SIZE - 3 |   M    |   M   | Cache used by Item Rate Meter/Level Meter for smoothing out (averaging) raw data measurements    |
| SIZE - 2      |   M    |   D   | Current measurement (item rate or level, as measured by Item Rate Meter/Level Meter)             |
| SIZE - 1      |   C    |   M   | Total count of items as computed by Item Counter                                                 |

**Secondary memory**

| Index          | Writes | Reads | Data                                                                                             |
|----------------|:------:|:-----:|--------------------------------------------------------------------------------------------------|
| 0 .. SIZE - 10 |   M    |  M,D  | Circular buffer holding quantities recorded over time (typically sampled at a frequency of 1 Hz) |
| SIZE - 9       |   M    |   D   | Foreground graph color                                                                           |
| SIZE - 8       |   M    |   D   | Background graph color                                                                           |
| SIZE - 7       |   M    |   D   | Start of the circular buffer (0)                                                                 |
| SIZE - 6       |   M    |   D   | End of the circular buffer, exclusive (SIZE - 7)                                                 |
| SIZE - 5       |   M    |   D   | Head of the circular buffer (i.e. latest value written)                                          |
| SIZE - 4       |  M,D   |  M,D  | Current minimum of recorded quantities                                                           |
| SIZE - 3       |  M,D   |  M,D  | Current maximum of recorded quantities                                                           |
| SIZE - 2       |   -    |   -   | Unused                                                                                           |
| SIZE - 1       |   -    |   -   | Unused                                                                                           |


### External memory access by schematics type

* Cache: used exclusively by the Item Rate Meter/Level Meter.
* Graph colors: written by the Item rate/level meters. The colors are chosen to correspond to the quantity being metered. 
* Circular buffer, including start index, end index and head position: values are written in by Item Rate Meter/Level Meter, and are read by Level Display.
* Current maximum/minimum:
  * Item Rate Meter/Level Meter: updates the minimum/maximum when a newly recorded quantity is larger than the current maximum (read/write access),
  * Level Display: reads the minimum/maximum and uses it to draw the graph. When drawing is finished, new minimum/maximum computed from accessed values are written back.
* Current measurement: written by Item Rate Meter/Level Meter. May be read e.g. by a Factory Monitor.
* Total count: written by Item Counter, read by Item Rate Meter.

