---
layout: post
title: Emacs Lisp 中的 quote 和 backquote
excerpt:
categories:
  - emacs
tags:
  - emacs
---

不仅 `emacs lisp` ， `quote` 在 `lisp` 系列的语言里面都存在，其主要用途告诉解释环境不要对被 `quote` 起来的元素求值，而保留其原本的样子。


# 求值序

其根本原因在于 `lisp` 的求值模型， `emacs lisp` 采用应用序 (applicative order) 来求值。应用序的特点是，求值时解释器优先将操作符和操作数都求值，然后再将整个表达式求值。在很多时候能够减少计算量。例如如下表达式：

```emacs-lisp
(defun square (n)
 (* n n))

(square (+ 2 5))

```

求值时首先对于 `square` 和 `(+ 2 5)` 分别求值， `square` 是一个函数， `(+ 2 5)` 得到 `7` ， 因此原式转化为 `(* 7 7)` 。另一种求值顺序是首先根据 `square` 的定义，将表达式展开为 `(* (+ 2 5) (+ 2 5))` ，再分别求值。可以看到这样会计算两次 `(+ 2 5)` 的值，这种求值方式叫做正则序 (normal order)。

如何验证这个说法呢？可以使用下面的表达式(这个例子来自于 SICP 的习题 1.5):

```emacs-lisp
(defun p () (p))

(defun test (x y)
  (if (= x 0)
      0
    y))
```

求值 `(test 0 (p))` 如果死掉了，那么一定是应用序，否则是正则序。

因此，采用应用序来求值，可以在一定程度上将计算量减少。但是同时也出现了一个问题，在一些操作是，不可能预先将所有参数都先求值，例如考虑如下的表达式， 依然使用上述的死循环 `p` ：

```emacs-lisp
(setq x (if (= 0 0) 0 (p)))
```

这个表达式是可以正常求值的，这表明， `if` 不是按照应用序来求值，而是根据需要再对参数展开。


# quote 的用处

除了 `if` 还有一些函数在应用时需要延迟求值，或者不需要求值，直接返回它本身。例如变量的赋值。

```emacs-lisp
(set x "y") ;; 直接这样是会报错的，因为 x 没有定义。
```

根据求值模型，上述表达式报错的原因是，求值时需要先知道 `x` 的值，而 `x` 正是我们要定义的，如果有一种方法， `x` 能代表 `x` 本身就好了，或者告诉求值器， `x` 就表示字面量 `x` ，不需要求值。

`quote` 函数就是用于这个场景， `(quote x)` 求值得到的是 `x` 。 有了这个机制，就可以完成赋值操作了 `(set (quote x) "y")` ， 表明将变量 `x` 赋值成 `"y"` 。由于 `quote` 用得太多，因此有了一个简写， 就是单引号， `\'x` 和 `(quote x)` 是等价的。对于 `set` 类函数，还可以简化的用 `setq` 来完成赋值。 `(setq x "y")` 和 `(set 'x "y")` 也是等价的。

顺便提一句， `quote` 已经是一个特殊形式，它是不能用来实现上一节提到的 `if` 或这里的 `setq` 的，要实现 `if` 而不更改求值序，可以使用 `lambda` 表达式。


# backquote 的用途

有了 `quote` ，看起来比较美好了，但是 `quote` 过于强势，对其引用起来的所有表达式都变成了它的字面形式。而有时，比如我们在配置 emacs 时，需要对 `quote` 里面的一些表达式做部分求值。比如下面的情况(这个例子来自于 `org mode` 的 `publish` 工程配置)：

```emacs-lisp
(setq my-org-export-dir "/path/to/dir")
(setq org-publish-project-alist
 '(("orgfiles"
   :base-directory my-org-export-dir
...)))
```

由于 `org-publish-project-alist` 本身是一个 `list` ， 因此在配置时候需要将其构造出来，按照字面来讲，用 `quote` 最合适了，但是我们需要将其中的 `my-org-export-dir` 替换成他的值。也就是说，在一个 `quote` 中我们要对某些变量进行部分求值，这时候 `backquote` 就有用了， `backquote` 也有一个简写，是 `` ` `` ，键盘上波浪号那个。一般情况下它和 `quote` 一样，如果需要部分求值，就在需要求值的表达式前面加上逗号。以下例子来自 emacs 自带的 elisp 说明。

```emacs-lisp
`(a list of (+ 2 3) elements)
    => (a list of (+ 2 3) elements)
'(a list of (+ 2 3) elements)
    => (a list of (+ 2 3) elements)

`(a list of ,(+ 2 3) elements)
          => (a list of 5 elements)
```

最后一个表明紧跟在逗号后面的表达式被求值了，其他都不求值。具体可以详细看 elisp 说明里面的 `backquote` 一节。之前的配置需求可以采用如下的方法解决：

```emacs-lisp
(setq org-publish-project-alist
 `(("orgfiles"
   :base-directory ,my-org-export-dir
...)))
```
