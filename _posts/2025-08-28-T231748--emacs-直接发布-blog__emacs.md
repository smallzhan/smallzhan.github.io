---
layout: post
title: emacs 直接发布 blog
comments: t
date: 2025-08-28
excerpt:
categories:
  - emacs
tags:
  - emacs
---

一直用 org mode 来写 blog, 之前的配置主要在 [这里](https://smallzhan.github.io/emacs/2021/08/08/T000000-利用-org-capture-写-blog__emacs.html)，这样配置后，每次发布，实际上还需要先发布当前文章到发布目录， 然后在这个目录用 `git add xxx, git commit, git push` 三连，感觉不是很方便，于是抽了点时间，整理了一个函数。

```emacs-lisp
(defun my-denote-publish-current-blog ()
    (interactive)
    (let* ((fname (buffer-file-name))
           (default-directory my-denote-blog-publish-dir)
           (blog-fname (concat (file-name-base fname) ".md")))
      (org-publish-current-file)
      (find-file-noselect
       (expand-file-name blog-fname default-directory))
      (with-current-buffer blog-fname
        (vc-git--call nil "add" blog-fname)
        (vc-git--call nil "commit" "-a" "-m" (concat "add blog " blog-fname))
        (vc-git--call nil "push" "origin"))

      (message (concat "publish " blog-fname "to github"))))

```

做的事情很直白，就是将刚刚写的文章发布一下，然后到发布目录，去执行 git 那几个操作，值得一提的是其中 `vc-git-call` 的应用，本来一开始使用的是 `magit-call-git`, 不过突然有一天使用时候，发现 `magit` 这个命令提示 `Symbol's function definition is void`, 才发现原来 `magit` 没有载入，所以就换成 vc 命令了。

另外，写 blog 的方式已经换成 denote 了，他不仅有更好的命名方式，也更容易使用。封装了一个命令

```emacs-lisp
(defvar my-denote-blog-dir (expand-file-name "blog/" denote-directory))
 (defun my-denote-blog ()
   "Create an entry in sub directory 'blog', while prompting for a title and keywords."
   (interactive)
   (let ((denote-date-format "%Y-%m-%d")
         (denote-id-format "%Y-%m-%d-T%H%M%S")
         (denote-id-regexp "\\([0-9-]\\{10\\}\\)-\\(T[0-9]\\{6\\}\\)")
         (my-denote-modify-blog-filename t))

    (denote
     (denote-title-prompt)
     (denote-keywords-prompt)
     'blog
     my-denote-blog-dir)))
```

其中那个怪异的 `my-denote-modify-blog-filename` 是因为 `denote` 对于 `jekyll` 形式的文件名识别日期有点问题，因此利用 `lexical-binding` 方式，临时去给 `date-to-time` 做了下 `advice`

```emacs-lisp
(defun my-modify-date-to-time-args (orig-args)
   (if my-denote-modify-blog-filename
       (let ((date (car orig-args)))
         (cons (s-replace "-" "" date) (cdr orig-args)))
     orig-args))

 (advice-add 'date-to-time :filter-args #'my-modify-date-to-time-args)
```
