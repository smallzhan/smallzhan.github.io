---
layout: post
title: jekyll github blog 标题转文件名函数
comments: true
excerpt: 
categories:
  - emacs 
tags:
  - emacs
---


# Jekyll 标题转文件名函数

在之前的一篇关于 org 写 github blog 的 [文章](https://smallzhan.github.io/emacs/2015/09/03/T000000-org-mode-写-github-pages__emacs.html)中，有一个函数是 `jekyll-make-slug` ，这个函数的作用是将标题字符串中的空格替换成 `-` ，从而作为文件名使用。这个函数的工作流程为，在开始写 `blog` 文章时，输入文章标题，自动根据文章标题来命名对应的 `org` 文件的名字。

```emacs-lisp
(defun jekyll-make-slug (s)
  "Turn a string into a slug."
  (replace-regexp-in-string
   " " "-" (downcase
            (replace-regexp-in-string
             "[^A-Za-z0-9 ]" "" s))))
```

原理非常简单，但是这个函数有一个问题：对于中文，转换会失败。如果文章标题全部是中文组成的，那么转换出来的结果是空字符串。

还有一个函数 `my-pages-start-post` ，这个函数实现了完整的 `blog` 文章命名规则。 `jekyll` 在发布的过程中，会将文件名前面的日期变成对应的文件夹，而真正生成在 `github` 可见的文章链接为日期后面的那个字符串，所以如果遇到中文，这个字符串就变成了空，这样会有问题，因此这个函数中，我在日期后面强行添加了一个 `p` ，这样即使遇到全中文的标题，至少也显示成 `p.html` 。

```emacs-lisp
(defun my-pages-start-post (title)
  "Start a new github-pages entry"
  (interactive "sPost Title: ")
      (let ((draft-file (concat jekyll-directory jekyll-posts-dir
              (format-time-string "%Y-%m-%d-p-")
              (jekyll-make-slug title)
              jekyll-post-ext)))
    (if (file-exists-p draft-file)
      (find-file draft-file)
      (find-file draft-file)
      (insert (format jekyll-post-template (jekyll-yaml-escape title))))))
```

但是，因为这个问题，导致如果同一天写两个全中文标题的 `blog` 的话，就会出现重复，而要解决这个重复，需要手动重命名这个文件，体验很不爽，所以，我同一天不会写两篇 `blog`&#x2026;。


# 一个解决方法

这个问题的解决思路是，对于全中文的标题，能根据中文能生成不同的文件名。如果有个函数可以将中文变成拼音，直接将拼音连起来就可以了。正好也在使用 `pyim` ，其中有这么 `2` 个函数

```emacs-lisp
(pyim-hanzi2pinyin STRING &optional SHOU-ZI-MU SEPARATOR RETURN-LIST
IGNORE-DUO-YIN-ZI ADJUST-DUO-YIN-ZI)

(pyim-hanzi2pinyin-simple STRING &optional SHOU-ZI-MU SEPARATOR RETURN-LIST
IGNORE-DUO-YIN-ZI ADJUST-DUO-YIN-ZI)
```

这两个函数的区别就是第一个函数会考虑多音字，我这个需求里面不需要。因此，我将上面的 `jekyll-make-slug` 函数进行了如下修改：

```emacs-lisp
(defun jekyll-make-slug (s)
    "Turn a string into a slug."
    (let ((s-pinyin (if (fboundp 'pyim-hanzi2pinyin-simple)
                        (if (<= (length s) 4)
                            (pyim-hanzi2pinyin-simple s nil " " nil)
                          (pyim-hanzi2pinyin-simple s t nil nil))
                      s)))
      (replace-regexp-in-string
       " " "-" (downcase
                (replace-regexp-in-string
                 "[^A-Za-z0-9 ]" "" s-pinyin)))))
```

如果文章标题比较长，会生成一个极其长的标题，比较难看。因此在标题长度大于 `4` 时，将生成汉字的首字母连起来作为标题。。。

这个方案不完美，主要是 `pyim-hanzi2pinyin-simple` 的问题，在中英文混合的时侯，生成的标题比较奇怪，不过考虑到原需求只要做一个不同标题的区分，也够用了。
