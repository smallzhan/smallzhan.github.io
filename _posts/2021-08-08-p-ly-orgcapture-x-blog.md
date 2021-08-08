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

之前在 emacs 里面一直使用 jekyll 写blog，采用的是一个独立的命令。这样也不是不好，不过看了 org-capture 之后一直再想如何将写 blog 集成到这个体系里面来，会带来一些方便，例如 clock，以及相应模板的设置，快捷键的绑定，以及一个narrow的界面。不过一开始研究一段时间，发现文件名这块很难放到模板里面，不过最近看到一个技巧，可以实现这个内容。 基本的技巧代码如下：

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

重点是模板中 `file` 后面的那个函数，在模板中，这个函数本身不能带参数，因此将参数利用 `setq` 保存起来，以后使用，在模板里面直接填充该变量，而函数执行时候可以调出一个界面来填充这个值。 因此就实现了从外界得到输入的方式。


# 更新jekyll文件名函数

之前采用pyim 的函数 `pyim-hanzi-to-pinyin-simple` 来做这个事情，但是这个函数有个问题，就是其中的英文会被吃掉，只剩下中文。另一方面，后来使用 rime 之后就把pyim 去掉了，所以这个函数也没使用了，最近研究了一下 `pinyinlib`，他里面包含了一个大的汉字表，因此写了两个小函数来实现这个功能，主要就是中英文混合的命令转换。

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

其中，`my-pinyinlib-convert-cc` 是将汉字转换成拼音首字母，而 `my-pinyinlib-convert-chinese` 就是将中文句子转换成首字母，其中的英文能够原样保留。
