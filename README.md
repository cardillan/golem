# golem

A collection of [Mindcode and Schemacode](https://github.com/cardillan/mindcode/tree/devel) scripts.

Individual program/schematics are stored in separate folders, hopefully accompanied by some readme. Source files for both Mindcode and Schemacode, as well as binary Mindustry schematic (`.msch`) files are present.

To compile Mindcode program into mlog, you can use the [web app](http://mindcode.herokuapp.com/). To compile Schematics files as they are here, you'll need [local installation of Mindcode](https://github.com/cardillan/mindcode/blob/devel/doc/syntax/TOOLS-IDE-INTEGRATION.markdown). 

## Table of contents

- **[Benchmarks](benchmarks)**. Collection of compiler benchmarks.
- **[Mandelbrot](mandelbrot)**. A schematics for drawing the famous Mandelbrot fractal using hyper-processors.
- **[Measurements](measurements)**. Collection of schematics for measuring and displaying quantities.
- **[Overdrive Dome Supply](overdrive)**. A simple schematics for supplying an overdrive dome directly from the core using a pair (two pairs if necessary) of units.
- **[Power Plant](power-plant)**. An impact reactor based power plant schematics. Can start from cold, supports up to 12 reactors and an overdrive projector/dome. 
- **[Production and consumption](production)**. Schematics for controlling or monitoring production or consumption of materials in Mindustry World. 
- **[Unit Transport](unit-transport)**. This is yet another implementation of units controlled by Mindustry logic to transport items, one that is automatic, self-correcting, highly configurable and providing detailed status information.
- **[Unit Transport (simple)](unit-transport-simple)**. A sample implementation of a simple unit transport, utilizing a separate processor for drone acquisition.