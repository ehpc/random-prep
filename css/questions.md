# How transform is faster than left/right

* No layout recalculation — transform doesn’t trigger reflow; geometry stays untouched.
* No repaint — content isn’t redrawn, only repositioned at the compositing stage.
* GPU compositing — moves the element to a separate GPU layer and manipulates it as a texture.
* Parallel processing — layout/paint on CPU, compositing on another thread or GPU.
* Matrix math — all transformations merge into one fast 4×4 matrix operation.
* Texture caching — the element’s bitmap (including text) is reused every frame.
* Font rendering reuse — text glyphs aren’t re-rasterized; the cached texture is scaled/rotated.
* Subpixel rendering skipped — disables complex AA for speed, using simpler GPU blending.
* Predictive optimization — will-change: transform lets the browser pre-allocate GPU resources.
