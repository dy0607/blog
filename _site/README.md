# dy0607's Blog

在这里记录一下目录结构，免得下次又找半天．．．

Plug-ins
===

Search
---

/Search 里面放了搜索插件的一堆东西.

/_includes/footer.html 中定义了搜索框的样式．

原插件的搜索按钮被改到了侧边栏上: /_includes/nav-links.html 

Highlightjs
---

代码高亮插件，官方网页<https://highlightjs.org/>，上面有各种样式

/styles 中存放了几个样式，在 /_layouts/post.html 中可以更改样式

Paginator
---

分页插件，好像就是官方提供的．．．

Gemfile 和 config 中定义了一些设置，比如每一页显示的文章数

/_includes/archives.html 中有换页的具体实现

Main Structure
===

Posts
---

/_posts 中存放了 markdown 格式的博文

/_layout/post.html 中加载了 Mathjex 等一堆 js/css

/_includes/post.html 是文章页面的样式

/_includes/comment-full.html 引用了Disqus评论（反正被墙了也没什么用）

Index
---

/_includes/head.html 中是首页大图那一块的东西

/_includes/archives.html 中定义了每篇文章在首页的显示格式．

其中使用了文章的 intro 属性，表示文章的内容概览．

stickie 属性为 true 的文章会置顶（似乎并没有什么卵用）

/_layout/content.html 中给首页加载了 MathJex

Categories
---

/_includes/categories.html 是分类浏览界面的样式，基本没动过．．．

Navbar
---

/_includes/navbar.html 是侧边栏的一些东西

/_includes/nav-links.html 中有侧边栏的链接

Footer
---

/_includes/footer.html 是页面底部的一些东西
