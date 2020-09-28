## ink和@import的区别

1. link 属于 HTML 标签，不只可以加载 CSS 文件，而 @import 是 CSS 提供的，只有导入样式表的作用

2. 导入的语法不同

   * link（链接式）语法为：

   ``` html
   <link rel="stylesheet" href="style.css" type="text/css" />
   ```

   * @import（导入式）语法为：

   ``` html
   <style>
   	@import url('style.css')
   </style>
   ```

3. 加载页面时，link 标签引入的 CSS 被同时加载，甚至多个 link 标签引入的 CSS 文件可以并行下载，而 @import 引入的CSS 将在页面加载完毕后被加载

4. link 作为 HTML 标签，不存在兼容性问题，而 @import 是 CSS2.1 才有的语法，故只可以在 IE5+ 才可以识别（当然，现在谁还适配 IE5 ）

5. 可以通过 JS 操作 DOM 插入 link 标签来改变样式，而 @import 是不能通过 JS 操作的

6. @import 一定要放在样式表的前面，不管是内部样式还是外部样式，如果放在末尾就会被浏览器忽略

