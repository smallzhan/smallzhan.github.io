---
layout: post
title: org-mode 和 org-babel 那些事(三)
comments: true
excerpt: 
categories:
  -  emacs
tags:
  -  emacs
---


# 缘起

近期一直在使用 `org mode` ，主要是作为任务管理工具，在纯文本的任务管理领域，org mode 实在特别好用，其他的工具基本上都难以望其项背。在参考了网上很多工作流之后也逐渐形成了自己的工作流，逐步实现任务跟踪和记录后，回顾起来看会感觉特别有意思。

前两天看到一篇论文，是专门讲 `org mode` 的，论文在[这里](https://www.jstatsoft.org/article/view/v046i03)。论文从文学编程（Literate Programming）和可重复性研究（Reproducible Research）两个方向探讨了一下 `org mode` 的功能，当然不论是文学编程还是可重复性研究，它们依赖的工具都是 `org-babel` 。读完论文后我觉得有必要从另一个角度来看待一下 `org mode` 和 `org-babel` 了。值得一提的是，他这篇投稿论文本身也是 `org-mode` 写成的。


# 文学编程

这个概念前面提过，它由两个部分组成，一个叫 `tangle` ， 一个叫 `weave` 。 如果把编程比喻成写文章， `weave` 是生成一个和源代码对应的文档，这个功能在 `org mode` 中原生支持，例如将代码块看作是代码，那其他所有的说明都是关于代码的说明，其他比较相近是类似 `sphinx` 这样的文档生成工具。而 `tangle` 就是将整篇文章里面的代码自动抽取出来，组合成一个可以编译的真正的程序。

实际上 `org` 源文档进行 `export` 成各种格式，就可以实现这里的 `weave` 过程， `tangle` 则需要通过 `org` 命令来执行。

就如同写文章不可能把所有内容写在一章里面，在文学编程中，也不可能把所有的代码都放在一个代码块里面。因此需要一个机制，将代码块拆分成很多个部分后，通过互相引用和插入，将其简单的组合起来。因此 `org-babel` 的代码块引入了 `:noweb` 关键字，将它设置为 `yes` 后，就可以使用代码块的引用了。具体方法是，对一个代码块通过 `+NAME:codename` 命名之后，在另一个代码块通过 `<<codename>>` 对其进行引用，实际上就是将这个代码块加入到当前的引用位置。


# 可重复性研究

这个词是从学术研究领域里面过来的，主要表达的是，在发表论文时，论文结果的实现需要很多的代码和环境，我们要保证在同样的代码和环境下，论文的结果是可重现的。这种情景还常常应用在教学上面，例如编程教学，确保学生根据相关的指示能进行完全一致的环境搭建和代码运行。通常的处理方式是，写一个很长的文档，一步一步告诉其他人怎么做，在不涉及交互时，这样其实也没什么问题，但是涉及到交互，可能就比较麻烦了。因为环境多种多样，交互式输出的结果可能不太一样。由于一个简单的不同可能会产生连锁反应，引起很多困扰。

于是就有了在文档中嵌入操作方式的一种做法，目前成功的应该是 `jupyter` ， 前身是 `ipython-notebook` ， 通过 `python` 代码和 `markdown` 文档将文字说明和代码放到一起，并且在后台启动一个 `kernel` ，当代码完成后，可以直接通过快捷键发送到 `kernel` 去运行，并把运行结果抓取回来，展示在文档上。最早 `jupyter` 只支持 `python` ， 在深度学习出现之后， `jupyter` 获得了大流行，各种深度学习的教程都采用对应的格式。后来 `jupyter` 经过发展也可以支持各种后端，包括 `R, julia` 等等。虽然支持了多种后端，但是目前好像还没看见能在同一个演示文档中组合多种语言环境。文档打开后，当前是什么环境就只能运行对应的语言。当然这个对于单一语言的教学其实不是太大的问题，遇到需要多种语言协作的情况就不是那么方便了。


# Org Mode

一般来说，文学编程和可重复性研究它们是两个不同领域的工具，很少有工具能将它们统一到一起，但是令人吃惊的是，在 `org mode` 里面它们是如此的和谐。

`org-babel` 实现的丰富的输出后端为 `weave` 提供了数不清的处理方式，而丰富的语言支持，则为 `tangle` 提供了各种各样互相运行和相关的结果。抛开文学编程不说，即使仅仅作为可重复性研究工具， `org-mode` 都可以将多种语言和环境集成起来，相互协作来完成一个复杂的任务。

用 `emacs-lisp` 作为中间件，利用命名代码块，将各代码块运行的结果保存起来，并在其他代码块之间传递，可以以接力的方式完成一件有意思的事情。下面看一下前面论文中给出的一个示例，该示例利用了不同的 `python` 代码块对数据进行处理，最后输出一个 `dot` 语言，使用 `org-babel` 生成了一个图片。

首先计算 `pascals-triangle` ：

    #+name: pascals-triangle 
    #+begin_src python :var n=5 :exports both :return pascals_triangle(5)
    def pascals_triangle(n):
        if n == 0: 
            return [[1]] 
        prev_triangle = pascals_triangle(n-1) 
        prev_row = prev_triangle[n-1] 
        this_row = map(sum, zip([0] + prev_row, prev_row + [0])) 
        return prev_triangle + [list(this_row)]
    
    return pascals_triangle(n) 
    #+end_src

在这个代码块上按 C-c C-c 就会得到如下结果，表格是 `org-mode` 中的呈现方式，实际上得到的就是一个二维数组。这个结果保存在代码块的 `name` 属性的变量中，即 `pascals-triangle` 。

    #+RESULTS: pascals-triangle
    | 1 |   |    |    |   |   |
    | 1 | 1 |    |    |   |   |
    | 1 | 2 |  1 |    |   |   |
    | 1 | 3 |  3 |  1 |   |   |
    | 1 | 4 |  6 |  4 | 1 |   |
    | 1 | 5 | 10 | 10 | 5 | 1 |

在下一个代码块中，我们将 `pascals-triangle` 赋值给 `pst` ， 然后生成 `dot` 语言的代码块，同样，我们将结果保存在 `pst-to-dot` 变量中。

    #+name: pst-to-dot 
    #+begin_src python :var pst=pascals-triangle :results output :exports both
    def node(i, j):
        return '"%d_%d"' % (i+1, j+1)
    
    def edge(i1, j1, i2, j2):
        return '%s--%s;' % (node(i1, j1), node(i2,j2))
    
    def node_with_edges(i, j):
        line = '%s [label="%d"];' % (node(i, j), pst[i][j])
    
        if j > 0:
            line += edge(i-1, j-1, i, j) 
        if j < len(pst[i])-1:
            line += edge(i-1, j, i, j) 
        return line
    
    pst = [list(filter(None, row)) for row in pst]
    
    print ('\n'.join([node_with_edges(i, j) 
                      for i in range(len(pst)) 
                      for j in range(len(pst[i]))])) 
    #+end_src

同样也是 `C-c C-c` 出现下面的结果，可以验证一下，确实是 `dot` 使用的语言。

    #+RESULTS: pst-to-dot
    
    "1_1" [label="1"];
    "2_1" [label="1"];"1_1"--"2_1";
    "2_2" [label="1"];"1_1"--"2_2";
    "3_1" [label="1"];"2_1"--"3_1";
    "3_2" [label="2"];"2_1"--"3_2";"2_2"--"3_2";
    "3_3" [label="1"];"2_2"--"3_3";
    "4_1" [label="1"];"3_1"--"4_1";
    "4_2" [label="3"];"3_1"--"4_2";"3_2"--"4_2";
    "4_3" [label="3"];"3_2"--"4_3";"3_3"--"4_3";
    "4_4" [label="1"];"3_3"--"4_4";
    "5_1" [label="1"];"4_1"--"5_1";
    "5_2" [label="4"];"4_1"--"5_2";"4_2"--"5_2";
    "5_3" [label="6"];"4_2"--"5_3";"4_3"--"5_3";
    "5_4" [label="4"];"4_3"--"5_4";"4_4"--"5_4";
    "5_5" [label="1"];"4_4"--"5_5";
    "6_1" [label="1"];"5_1"--"6_1";
    "6_2" [label="5"];"5_1"--"6_2";"5_2"--"6_2";
    "6_3" [label="10"];"5_2"--"6_3";"5_3"--"6_3";
    "6_4" [label="10"];"5_3"--"6_4";"5_4"--"6_4";
    "6_5" [label="5"];"5_4"--"6_5";"5_5"--"6_5";
    "6_6" [label="1"];"5_5"--"6_6";

我们取出 `pst-to-dot` 变量并赋值给 `pst-vals` ，并且组合成合法的 `dot` 格式程序， 然后 `C-c C-c` 调用 `dot` 画图。

    #+name: pst-to-fig 
    #+headers: :file pascals-triangle.png :cmdline -Tpng
    #+begin_src dot :var pst-vals=pst-to-dot :exports both
    graph {
    $pst-vals
    } 
    #+end_src
    
    #+RESULTS: pst-to-fig

最后就生成了如下的图片

![img](https://smallzhan.github.io/images/pascals-triangle.png)
