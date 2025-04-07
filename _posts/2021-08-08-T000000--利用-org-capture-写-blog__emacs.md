---
layout: post
title: 利用 org-capture 写 blog
comments: true
excerpt: 
categories:
  -  emacs
tags:
  -  emacs
---


# org-capture 的优势以及技巧

之前在 emacs 里面一直使用 jekyll 写blog，采用的是一个独立的命令。在看了 org-capture 之后一直在想如何将写 blog 集成到 capture 这个体系中来，这样会带来一些体验上的便利，例如可以跟踪写的时间 &#x2013; clock in/out 功能，以及模板的可选择设置，快捷键的绑定，以及一个独立的 narrow 界面。开始研究了一段时间，发现 template 的定义里面很难将动态输入的文件名当做最终的文件目标。最近看到了一个技巧，可以实现这个功能。

基本的技巧代码如下：

```emacs-lisp
(defun my/generate-org-note-name ()
  (setq my-org-note--name (read-string "Name: "))
  (setq my-org-note--time (format-time-string "%Y%m%d%H%M%S"))
  (expand-file-name (format "%s-%s.org" my-org-note--time my-org-note--name) "~/tmp"))

(setq org-capture-templates
  '(("n" "note" plain 
     (file my/generate-org-note-name)
     "%(format \"#+TITLE: %s\n#+STAMP: %s\n\" my-org-note--name my-org-note--time)")))
```

重点是模板中 `file` 后面的那个函数，虽然在模板中，这个函数不能带参数，但是函数中可以通过 read-string 获取用户输入的文件名，并将文件名利用 `setq` 保存起来，方便以后的使用。这样就实现了动态的获取文章名字的方式。


# 更新jekyll文件名函数

jekyll中的文件名采用的是中文的首字母缩写，之前采用pyim 的函数 `pyim-hanzi-to-pinyin-simple` 来实现，不过这个函数有个问题，其中的英文会被吃掉，只剩下中文。如果是纯英文的标题，就完全没有名字了。另一方面，使用 rime 之后就把 pyim 去掉了，所以这个函数也无法使用了。最近研究了一下 `pinyinlib` ，其中包含了一个很大的汉字到拼音的表，因此写了两个辅助小函数，来实现中英文混合时的标题名转换。

```emacs-lisp
(defun my-pinyinlib-convert-cc (cc)
   "convert character cc to pinyin if cc is chinese character"
   (let ((i 0)
         (str (char-to-string cc)))
     (if (< cc 255) ;; FIXME: english character...
         str
       (catch '__
         (dolist (l pinyinlib--simplified-char-table)
           (if (string-match (char-to-string cc) l)
               (throw '__ (char-to-string (+ 97 i)))
             (cl-incf i)))))))

 (defun my-pinyinlib-convert-chinese (chinese)
   "convert chinese string to pinyin"
   (mapconcat (lambda (c) (my-pinyinlib-convert-cc c))
              chinese ""))

 (defun jekyll-make-slug (s)
   "Turn a string into a slug."
   (unless (featurep 'pinyinlib)
     (require 'pinyinlib))
   (let ((s-pinyin (my-pinyinlib-convert-chinese s)))
     (replace-regexp-in-string
      " " "-" (downcase
               (replace-regexp-in-string
                "[^A-Za-z0-9 ]" "" s-pinyin)))))
```

其中， `my-pinyinlib-convert-cc` 是将汉字转换成拼音首字母，而 `my-pinyinlib-convert-chinese` 就是将中文句子转换成首字母，其中的英文能够原样保留。简单使用了 `(< cc 255)` 来判断是否英文字母。
