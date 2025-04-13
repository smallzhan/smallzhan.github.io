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

对于 `org-mode` 来说，除开其作为时间管理的部分之外，它最佳的功能其实是格式转换。相对之下，其他所有的文本格式，转换感觉都比较难用。如果说 `org-mode` 站在顶点，那山下包括 `markdown` ， `reST` ，还有一些封闭格式自认不错的其他标记语言。 `markdown` 似乎只能较好的输出 `html` 格式，其他格式都需要一些技巧，而 `reST` ，我一直没整明白过。

`org-mode` 的输出，支持的格式包括：

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

`ascii` 模式可能过于简单，几乎就是原文档。常见的格式包括 `html` ， `md` ， 还有 `latex` 等， `latex` 可以比较方便的转化成格式化的 `pdf` 。还有 `odt` ，全称是 `open document text` 。可能不大常看到，它有个变种比较强大，就是微软的 `doc` 格式，即大名鼎鼎的 `word` 。如果文档中没有太复杂的格式， `odt` 的输出也比较通用了。 `texinfo` 和 `man` 常见于 `unix-like` 系统，基本是终端显示里面的老大哥。这些只是自带的，如果算上 `melpa` ，输出格式就更多了，包括 `ox-epub` ， `ox-mediawiki` ， `ox-pandoc` 等等。以前我写文档喜欢用 `latex` 生成 `pdf` ，但一段时间之后，常常只有 `pdf` 文档保存下来，而对应的 `latex` 源码常丢。而且因为这个格式太专业，也没有办法从 `pdf` 文件逆向生成回来，于是后来简单配置了下 `org-mode` 到 `latex` 再到 `pdf` 输出这样的链路，发现 `org-mode` 方便得一塌糊涂。在 `org-mode` 中配合 `cdlatex` ，可以直接实现数学公式的实时预览。

当然 `org-mode` 唯一的缺点是，和 `emacs` 存在强绑定，但是对于习惯了 `emacs` 的用户，这反而是优点。


# Org-babel 助力输出

配置 `org-babel` 之后， `org-mode` 在输出方面上能玩的花样就更多了。例如，写文章时，需要用到一段程序的输出结果。一般情况下，需要另开一个编辑器写程序，然后编译，运行。再把输出，贴到文档中。至少要离开当前的编辑环境，去外面绕一圈。

在带有 `org-babel` 支持的 `org-mode` 中，只需要使用 `#+begin_src` 和 `+end_src` 包裹这个代码块，就可以直接在当前环境下写程序。而且他可以支持各种变成环境，具体的环境可以在 `melpa` 上看那些 `ob-` 开头的扩展，基本上支持了所有主流非主流的编程语言。

例如需要一个 `python` 程序，可以使用下面的格式

    #+BEGIN_SRC python :results output  
    print("hello, world 1")
    #+END_SRC

在这个代码块上用下 `C-c C-c` 快捷键，立马结果就出来，整个过程中都不需要从 emacs 里面出去。结果会放在一个 `#+RESULTS:` 的标签里面

上面的程序会显示成如下结果

    #+RESULTS:
    : hello, world 1

如果 `SRC` 代码块太长，在编辑时可以使用 `narrow` 模式，所谓 `narrow` 是，把当前编辑的文本中的其他部分全部隐藏起来，只保留 `SRC` 部分。这样在视觉和交互上就像开了一个专门的 `ide` 在写代码一样。而且，完全与写代码的体验一致，包括语法高亮，智能补全等功能，都能生效。

简单配置一下，还可以代码块中生成需要的图片，然后将图片输出到当前文件，在下面的文档中直接引用这个图片。这个功能，在 `latex` 输出的时特别顺畅，作图，插图，用一个非常符合直观的方式将这些流程有机集成了起来。

目前其他能做到类似功能的只有 `python-notebook` ，即后来的 `jupyter` 。随着 `python` 在人工智能的领域的大展拳脚，以 `ipynb` 形式出现的学习材料越来越多。可惜的是，因为是一个网页版本，受到浏览器的限制，无法实现完整的编辑功能，包括编辑代码需要的高亮，补全等支持也差强人意。可以说在编辑特性上完全与 `emacs` 无法比较。而且 `jupyter` 支持的语言也比较少，无法与庞大的 `emacs` 生态相提并论。
