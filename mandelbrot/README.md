# Mandelbrot

A schematics displaying the Mandelbrot-set based fractal in a display. Computed in parallel using 25 hyper-processors.

Allows navigation inside the picture (movement along the x/y axes, zoom in, zoom out), switching to the Julia set corresponding to the point in the centre of the display, bookmarks for storing/loading positions, and five different coloring schemes.

Given enough resources, can be built and used in normal gameplay (outside sandbox). Using an overdrive dome is essential.

Due to unresolved issues with parallel drawing operations, the display might not be updated when it is not visible on the screen, meaning the final image won't be complete.

![Screenshot of the schematics in action](Mandelbrot.png)