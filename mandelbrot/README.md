# Mandelbrot

![Screenshot of the schematics in action](Mandelbrot.png)

A schematics displaying the Mandelbrot-set based fractal in a display. Computed in parallel using 24 hyper-processors.

Allows navigation inside the picture (movement along the x/y axes, zoom in, zoom out), switching to the Julia set corresponding to the point in the centre of the display, bookmarks for storing/loading positions, and five different coloring schemes.

Given enough resources, can be built and used in normal gameplay (outside sandbox). Using an overdrive dome, which is not included in the schematics, is essential.

Since Mindustry doesn't redraw displays which aren't shown on the screen, portions of the fractal computed while the display was not visible are not drawn. The only way to fix this is to pause the computing while the display is out of view, and it is still being worked on.
