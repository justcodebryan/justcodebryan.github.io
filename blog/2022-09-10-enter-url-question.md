---
slug: 浅谈输入URL到显示网站部分问题
title: 浅谈输入URL到显示网站部分问题
authors: [justcodebryan]
tags: [browser-core]
---

1. 为什么`JS`的解析会阻塞`DOM Tree`的解析

   因为`JS`的解析运行被默认为有可能使用`document.write`方法, `document.write`方法会向文档流里面写入字节, 这样的话, 会强制刷新剩下的文档

   所以为了避免重复操作, 就将剩下的`dom`解析给阻塞了

   而且在这时的`js`脚本能够操作前面已经解析完成的`dom`元素, 而不能操作后面还未解析的`dom`元素, 因为还未解析出来, `dom`元素还不存在

   `document.write`使用流程:

   1. `document.open`
   2. `document.write`
   3. `document.close`

   从另外一个角度理解, 当浏览器在解析 DOM 树的时候, 已经调用了`document.open`了, 那么在这种情况下, 使用`document.write`可以直接在文档中间的位置写入内容, 当文档解析完了触发`documentloaded`的时候, 浏览器会调用`document.close`。这个时候相当于写入文件完成并且管道关闭了，如果还要再进行`document.write`这个操作的时候, 需要重新调用`document.open`。因为是系统自动调用的， 系统不知道哪个`document`，所以就会重新生成一个空白的`document` , 并且把`document.write`里面要写入的内容写进去, 并且覆盖掉原来的`document`, 这样就会造成`documentloaded`之后调用`document.write`会将整个页面都覆盖掉。

2. JS script 不同标签的行为(async, defer)

   async - 异步加载, 同步执行

   defer - 异步加载, onload 之后执行

3. CSSOM 解析会阻塞 DOM Tree 的解析或者 JS 的运行吗

这里共有两种情况：

1. 如果 CSS 外链标签在文件的最上方, 异步下载解析, 但是文件中间有一个`script`标签, 当整个文件解析到了 script 标签的时候, 假定 CSS 标签还未下载完成, 如果 script 标签里面有 get ComptuedStyle 的操作, CSS 的解析就会阻塞 JS 的执行, 直到 CSS 解析完成了以后, 获取了正确的样式, 这样计算出来的结果才是正确的。

```html
<html>
  <head>
    <link src="./index.css" />
    <title>test app</title>
  </head>
  <body>
    <script src="./test.js"></script>
  </body>
</html>
```

2. 前面的情况同上, 唯一不同的点就是, script 标签里面没有做 get computed style 的操作。这时，就和一般的操作是一样的。

3. 浏览器并行下载的最大数量

   各个浏览器的并行下载数量不同, 之前的最大是 6 个

   ```markdown
   Firefox 2: 2
   Firefox 3+: 6
   Opera 9.26: 4
   Opera 12: 6
   Safari 3: 4
   Safari 5: 6
   IE 7: 2
   IE 8: 6
   IE 10: 8
   Edge: 6
   Chrome: 6
   ```

4. 为什么需要 document.write 方法
   以下内容摘自 Stack Overflow
   When document.write() is executed after page load, it replaces the entire header and body tag with the given parameter value in string.

   - document.write (henceforth DW) does not work in XHTML
   - ~~DW does not directly modify the DOM, preventing further manipulation~~ _(trying to find evidence of this, but it's at best situational)_
   - DW executed after the page has finished loading will overwrite the page, or write a new page, or not work
   - DW executes where encountered: it cannot inject at a given node point
   - DW is effectively writing serialised text which is not the way the DOM works conceptually, and is an easy way to create bugs (.innerHTML has the same problem)

   The only seem appropriate usage for document.write() is when working third parties like Google Analytics and such for including their scripts. This is because document.write() is mostly available in any browser. Since third party companies have no control over the user’s browser dependencies (ex. jQuery), document.write() can be used as a fallback or a default method.(兼容性问题, 当需要加入 google analytics 等的脚本, 这是最容易加入依赖的方法)

5. render tree 的行为, 如果 CSS 发生了改变以后

   repaint(重绘)和 reflow(重排)

6. 阻塞渲染和阻塞解析

   阻塞渲染: CSS 的解析会阻塞渲染 -> 基于不要让用户看到没有 CSS, 即没有样式的页面, 但是也会有例外的情况, 有可能很久没有返回 CSS, 这时, CSS 一直无法解析, 那就有可能出现没有样式的页面, 但是最后接收到了页面 CSS 就会让重新刷新

   阻塞解析: JS 的运行会阻塞 DOM 树解析

   并行下载: 最大数量下载量 -> 下载的时间和时机看代码的位置和配置
