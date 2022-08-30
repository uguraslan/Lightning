# Runtime Configuration


When you initialize your application, you configure the behavior of Lightning at runtime. For example, you can define the use of WebWorkers or set the size of the canvas.


For that purpose, you use *Lightning configuration options* at Application and / or Render Tree level, where you define the configuration options for the Render Tree in the `stage` sub object.


For example, you might specify the following configuration options:


```
const options = {debug: true, stage: {w: 1920, h: 1080, clearColor: 0xFF000000}}
const App = new MyApp(options);
```

## Application Configuration Options

| Name | Type | Default | Description |
|---|---|---|---|
| `debug` | Boolean | false | Shows changes to the focus path for debug purposes |
| `keys` | Object | Object | A custom [key map](../HandlingInput/RemoteControl/KeyHandling.md#key-mapping) |





## Stage Configuration Options

| Name | Type | Default | Description |
|---|---|---|---|
| `canvas` | HTMLCanvasElement | null | If specified, the canvas to be reused (default: create a new canvas) |
| `context` | Object | null | If specified, the WebGL / Canvas2D context to be used |
| `w` | Number | 1920 | Stage width in pixels |
| `h` | Number | 1080 | Stage height in pixels |
| `precision` | Float | 1 | Global stage scaling (see details below) |
| `memoryPressure` | Number | 24e6 | Maximum GPU memory usage in pixels (see details below) |
| `clearColor` | Float[] | [0,0,0,0] | Background color in ARGB values (0 to 1) |
| `defaultFontFace` | String | sans-serif | Font face for text rendering |
| `fontSharp` | Object, Boolean | { precision:0.6666666667, fontSize: 39 } | Determine when to apply gl.NEAREST to TEXTURE_MAG_FILTER |
| `fixedDt` | Number | 0 (auto) | Fixed time step per frame (in ms) |
| `useImageWorker` | Boolean | true | By default, use a Web Worker that parses images off-thread (web only) |
| `autostart` | Boolean | true | If set to *false*, no automatic binding to  `requestAnimationFrame` |
| `canvas2d` | Boolean | false | If set tot *true*, the Render Engine uses canvas2d instead of WebGL (limitations apply, see details below) |
| `readPixelsBeforeDraw` | Boolean | false | If set to *true*, forces the Render Engine to readPixels before drawing, turning the Render pipeline to synchronous (this helps with flickering artifacts on certain devices). Note this will affect performance! |
| `readPixelsAfterDraw` | Boolean | false | If set to *true*, forces the Render Engine to readPixels after drawing turning the Render pipeline synchronous (this helps with flickering artifacts on certain devices). Note this will affect performance! |
| `forceTxCanvasSource` | Boolean | false | If set to *true*, forces the Render Engine to use the canvasSource over getImageData for canvas textures (this helps with text generation on certain devices). |
| `pauseRafLoopOnIdle` | Boolean | false | If set to *true* will stop the Render Engine from calling `RequestAnimationFrame` when there are no stage updates. |




## Downscaling

> We advise you to always develop your TV App in a **1080p** coordinate system (the Lightning default).


Assume that you have created an App for 1080p quality, so you have used a 1920x1080 coordinate system to position all of the content. However, you have found out that the App needs to be displayed in a 1280x720 resolution.


You can use the `precision` property to perform a  *global rescale* of the coordinate system. For example, if you specify `precision: 2/3`, the 1920 x-position will be mapped to the 1280-output pixel coordinate.


As a result, the text and off-screen textures are rendered at a *lower resolution* by the [Render Engine](../RenderEngine/index.md), which increases quality (less pixel interpolation) and reduces memory usage.


Downscaling with the `precision` option generally works well. But keep in mind that WebGL rasterizes as *pixel boundaries*, so when it uses a line width of 2 in 1080p quality, it may render at either 2px or 3px in 720p (depending on the rendered pixel offset). If you encounter such problems, you have to set the sizing at a multiple of 3.


## FontSharp

By default we apply `gl.LINEAR` to texture `TEXTURE_MAG_FILTER` parameter. This can lead to a more blurry font
when we try to render smaller fonts on lower precisions. By changing the `fontSharp` stage setting you can adjust the behaviour:

```js
fontSharp: {
    precision:0.6666666667,
    fontSize: 39
}
```

means: set texture magnification filter (gl.TEXTURE_MAG_FILTER) to `gl.NEAREST` when the font-size of our current text texture is
lower or equal to 39 and our render precision is lower or equal to 0.6666666667.

```js
fontSharp: false
```

Will disable it completely and will use `gl.LINEAR` as texture magnification filter.


## GPU Memory Tweak


This defines the total number of pixels that may be allocated in GPU memory, which allows you to tweak the amount of GPU memory that your Lightning App can use.


If this amount is reached, Lightning only performs an *unused texture cleanup*. Ideally, this situation does not occur often.

> A single pixel uses between 4 and 6 bytes of GPU memory.

## Limitations of `Canvas2D`


If WebGL is not available or if Canvas2D is set to *false*, Lightning uses *Canvas2D* for rendering output.


Note that some functionality (such as WebGL-only shaders) will not work (the default shader will be used instead).


Another limitation applies to  *colorizing textures*. You can safely colorize rectangle textures and even provide (linear-only) gradients. However, other textures such as text and images require the engine to create an *offscreen* colorized texture, where gradients are not supported. This is expensive in terms of CPU and memory, but if used sparingly to color text or tint images one time, it works fine in practice.