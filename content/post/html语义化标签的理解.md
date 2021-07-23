---
title: "Html语义化标签的理解"
date: 2021-07-03T14:59:06+08:00
draft: true
---

### HTML语义化是什么？

语义化是指根据内容的结构化（内容语义化），选择合适的标签（代码语义化），便于开发者阅读和写出更优雅的代码的同时，让浏览器的爬虫和机器很好的解析。

### 为什么要语义化？

- 有利于SEO，有助于爬虫抓取更多的有效信息，爬虫是依赖于标签来确定上下文和各个关键字的权重。
- 语义化的HTML在没有CSS的情况下也能呈现较好的内容结构与代码结构
- 方便其他设备的解析
- 便于团队开发和维护

---

### html 语义化标签

HTML为网页文档内容提供上下文结构和含义。对于HTML体系而言，Web语义化是指使用语义恰当的标签，使页面有良好的结构，让页面元素有含义，便于被浏览器、搜索引擎解析、利于SEO。通常我们所说的HTML应该是完全脱离表现信息的，其中的标签应该都是语义化地定义了文档的结构。 代码示例：

```html
<html>
    <body>
        <article>
            <header>
                <h1>h1 - WEB 语义化</h1>
            </header>
            <nav>
                <ul>
                    <li>nav1 - HTML语义化</li>
                    <li>nav2 - CSS语义化</li>
                </ul>
            </nav>
            <section>
                section1 - HTML语义化
            </section>
            <section>
                section2 - CSS语义化
            </section>
            <time datetime="2021-07-03" pubdate>time - 2021年07月03日</time>
            <footer> footer </footer>
        </article>
    </body>
</html>

```

html语义化标签包括 body, article, nav, aside, section, header, footer, hgroup, 还有 h1-h6 address等。

下面来简单介绍下常用的html语义化标签

#### header 元素

>header代表“网页”或者“section”的页眉，通常包含h1-h6 元素或者 hgroup, 作为整个页面或者一个内容快的标题。也可以包裹一节的目录部分，一个搜索框，一个nav，或者相关logo。

代码示例：

```html
    <header>
        <hgroup>
            <h1>网站标题<h1>
            <h2>网站副标题</h2>
        </hgroup>
    <header>

```

注意事项：

1. 可以是“网页”或者任意“section”的头部部分
2. 没有个数限制
3. 如果hgroup或者h1-h6自己就能工作得很好，那么就没必要用header。

---

#### hgroup 元素

`hgroup` 元素代表“网页”或“section”的标题，当元素有多个层级时，该元素可以将`h1`到`h6`元素放在其内，譬如文章的主标题和副标题组合

代码示例：

```html
<hgroup>
    <h1>这是一个主标题</h1>
    <h2>这是一个副标题</h2>
</hgroup>

```

注意事项：

1. 如果只需要一个h1-h6标签就不用hgroup
2. 如果有连续多个h1-h6标签就用hgroup
3. 如果有连续多个标题和其他文章数据，h1-h6标签就用hgroup包住，和其他文章元数据一起放入header标签

---

#### footer 元素

`footer`元素代表“网页”或任意“section”的页脚，通常含有该节的一些基本信息，譬如：作者，相关文档链接，版权资料。如果`footer`元素包含了整个节，那么它们就代表附录，索引，提拔，许可协议，标签，类别等一些其他类似信息。

代码示例：

```html
<footer>
    COPYRGHT@Alen
</footer>

```

注意事项：

1. 可以是“网页”或者任意“section”的底部部分
2. 没有个数限制，除了包裹的内容不一样，其他跟header类似

---

#### nav 元素

nav 元素代表页面的导航链接区域。用于定义页面的主要导航部分。 代码示例：

```html
<nav>
    <ul>
        <li>HTML语义化</li>
        <li>CSS 语义化</li>
    </ul>
</nav>


```

侧边栏上目录、面包屑导航、搜索样式、或者下一篇上一篇文章我们可能会想要用到nav，但是事实上规范上说nav只能用在页面主要导航部分上。页脚区域中的链接列表，虽然指向不同网站的不同区域，譬如服务条款，版权页等，这些footer元素就能够用了。

注意事项：

1. 用于整个页面的主要导航部分，不适合就不要用nav元素了

