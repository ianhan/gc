# gc

This library is something I've used over the years for microcontroller and FPGA graphics - LCDs, OLEDs, FPGA video experiments, and off-device hardware like DisplayLink. It supports hardware acceleration and doesn't require an on-device framebuffer. It's literally called "graphics context" because I just needed graphics the first time I used it. It probably looks like GDI, that is not a coincidence.

It supports fills, lines, rectangles, circles, flood fill, gradients, blits, stretch blits, transformed blits, alpha/color-key blits, palettes, and color bitmap fonts. The pixel format is flexible and you don't even have to have some color channels (for monochrome/bicolor displays).

The library is built around a GCDEVICE callback table. The same drawing code can draw into system memory or into a device backend. Some backends are included, such as the DisplayLink backend in tusb_libdlo.

This is a spare-time "I needed some bits" graphics library and I make no guarantees. It probably has dragons in it, but PicoGraph uses it so I've open sourced it.