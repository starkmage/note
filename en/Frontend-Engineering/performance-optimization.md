Reference article: https://segmentfault.com/a/1190000022205291

## JavaScript Section

1. Place scripts at the bottom, or add defer/async attributes
2. Route lazy loading
3. Image optimization
   1. Image lazy loading
   2. Reduce image quality
4. Debouncing and throttling
5. Avoid memory leaks
6. Reduce unnecessary DOM access
7. Event delegation
8. Use requestAnimationFrame for visual changes

## CSS Section

1. Place in head, preferably use link tags

2. For large CSS files, use media queries to split them into multiple CSS files for different purposes, so specific CSS files are only loaded in specific scenarios

3. Reduce repaints and reflows, for example by using GPU acceleration techniques like transform and opacity

4. Reduce CSS selector complexity

5. Use flexbox instead of older layout models

6. Implement animations using transform

   When using left/top for position changes, animation nodes and Document are rendered in the same GraphicsLayer, causing continuous repaints of the entire Document. Using transform places the animation node in an independent composite layer for rendering, so animations don't affect other layers. Additionally, animations run completely on the GPU, making them smoother compared to CPU processing layers and sending them to the graphics card for display.

   https://juejin.cn/post/6844903966573068301

## Bundling Section

1. Use externals to exclude third-party packages
2. Code splitting, extract common files
3. Image compression (imageWebpackPlugin)
4. Code compression, TerserPlugin
5. gzip compression (CompressionWebpackPlugin)
6. happPack to accelerate bundling
7. TreeShaking

## Network Section

1. Reduce HTTP requests
2. Use HTTP/2
3. Configure caching appropriately
4. CDN acceleration

## Others

1. Use server-side rendering
2. **Optimize moderately**