---
layout: post
title: Org mode 和 Org-babel 的那些事(二)
comments: true
excerpt: 
categories:
  - emacs  
tags:
  -  emacs
---


# Org mode 的输出

其实对于 orgmode 来说，除开其作为 gtd 的部分之外，它最佳的部分其实是输出。其他所有的文本格式，输出都不要太烂。这点，orgmode 可以站在顶点，对一票包括 markdown，reST，还有一些自己封闭自认不错的标记语言等等踩在脚下。markdown 吧，可能就能输出一个 html，其他的绕好久，reST，貌似没整明白过。

先看裸输出吧，支持了这么一票输出

<p class="verse">
• ascii (ASCII format)<br />
• beamer (LaTeX Beamer format)<br />
• html (HTML format)<br />
• icalendar (iCalendar format)<br />
• latex (LaTeX format)<br />
• md (Markdown format)<br />
• odt (OpenDocument Text format)<br />
• org (Org format)<br />
• texinfo (Texinfo format)<br />
• man (Man page format)<br />
</p>

好吧，ascii 可能过于简单，没啥人用，常见的也就是 html, md, 还有 latex, 提 latex 是因为它背后就是 pdf。哦，还有 odt，看起来没啥人知道，但是，它有个变种比较强大，就是微软的 doc，大名鼎鼎的 word。里面不塞太复杂的格式的话，其实也通用了。texinfo 和 man 那也是 unix-like 系统里面的老大哥。这些还是自带的，算上 melpa 的话，那啥 ox-epub 啊，ox-mediawiki 啊，ox-pandoc 等等的都跑出来了。以前我写点啥玩意喜欢用 latex 搞成 pdf，但是格式是有了，源码常丢，而且因为太专业，也不好各种转换，后来简单配置了下 latex 输出之后，发现 org-mode 方便得一塌糊涂。

当然 org-mode 的唯一缺点就是绑定 emacs 了。

配合了 cdlatex 的 orgmode，数学公式写一下，实时就给预览了。


# Org-babel 助力输出

加上 org-babel 之后，orgmode 在输出上就更能玩花样了。比方把，一般写文章，需要用一段程序跑出一个什么结果。一般，嗯，总不能在写文章的编辑器直接写这个程序吧，那再开个写程序的编辑器，写完跑一下，看看结果，嗯，拿过来，贴过来。至少要去外面绕一圈咯。

在配合有 org-babel 的 orgmode 里面，完全就不用这样干了，begin 一个代码块，直接写了。 能支持的都用 ob- 开头，去 melpa 看一下， 吓到了，各种主流非主流的语言都有了。

<p class="verse">
#+BEGIN<sub>SRC</sub> python :results output<br />
print("hello, world 1")<br />
\#+END<sub>SRC</sub><br />
</p>

在这个代码块上用下 `C-c C-c` 快捷键，立马结果就出来，整个过程中都不需要从 emacs 里面出去。结果会放在一个 `#+RESULTS:` 的标签里面

上面的就是这样的结果

<p class="verse">
#+RESULTS:<br />
: hello, world 1<br />
</p>

然后呢, 如果 SRC 里面的代码太长，也有办法，就是 narrow 一下，所谓 narrow 就是，把这个 buffer 里面的其他部分全部隐藏起来，只保留这个 SRC 的部分，就像真在写代码一样。而且，是真像在写代码一样，高亮啊，补全啊，都存在。

简单配置一下，还可以代码块里面写画图的程序，然后图片输出到文件，在下面的代码里面直接引用这个图片。这个功能，在用 latex 输出的时候不要太爽，作图，插图，一把给搞定了。

目前其他能做到类似功能的只有 python-notebook，也就是后来的 jupyter，jupyter 随着 python 人工智能的兴起，往往作为一个教学文件。可惜的是，它牺牲了编辑代码的体验，因为是一个网页，并不是完整的编辑器，所以编辑特性太差了。
