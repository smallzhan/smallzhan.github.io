---
layout: post
title: emacs org-mode 写 github-pages
excerpt:
categories:
  -  emacs
tags:
  -  emacs
---


# mac 下的 emacs

emacs-mac 应该算是 mac 下最好的 emacs 了， 做了很多的增强，例如把 M 键映射到Command ， 并且还支持触摸板的手势滑动前进后退， 以及启动时直接从 shell 中读取 =$PATH=。可以直接 homebrew 安装。

```sh
brew tap railwaycat/emacsmacport
brew install emacs-mac --with-official-icon
```

如果不用 `--with-official-icon` 的话那个图标丑爆了。。。


# emacs 的 org mode 写 git pages

主要是写 org mode 习惯了，这个确实是 emacs 中的大神器。

写 github pages 的配置主要参考[这里](http://www.gorgnegre.com/linux/using-emacs-orgmode-to-blog-with-jekyll.html) ，不过他把简单问题复杂化了，我在其基础上简化了配置。不再输出直接输出 html 文件，而是输出为 github 可以渲染的 markdown 格式。可以使用 ox-gfm 这个专门为 github pages 优化过的输出器，这个输出器可以在 melpa 直接安装。

首先定义一个输出工程的函数给 project 用。

```emacs-lisp
(require 'ox-gfm)

(defun org-gfm-publish-to-markdown (plist filename pub-dir)
  "Publish an org file to MarkDown with GFM.

    FILENAME is the filename of the Org file to be published.  PLIST
    is the property list for the given project.  PUB-DIR is the
    publishing directory.

    Return output file name."
  (org-publish-org-to 'gfm filename ".markdown"
                      plist pub-dir))
```

因为 jekyll 默认会渲染工程目录 \_posts 下面的 markdown 文件，目录结构大概如下：

    |blog
    |  |_posts
    |     |-- 2015-09-03-my-first-post.org
    |
    |
    |jekyll
    |   -- _config.yml
    |   -- _layouts
    |      |-- default.html
    |      `-- post.html
    |   -- _posts
    |      |-- 2015-09-03-my-first-post.markdown
    |
    |   -- |_site
    |   -- |_includes
    |   -- index.html

根据这个结构，可以如下配置 post 的 project。在 org publish 的时候输出整个 blog 目录中所有的 org 文件，当然，没更改的文件不会重新输出。

```emacs-lisp
(setq org-publish-project-alist
 `(("gitpages" ;; settings for cute-jumper.github.io
   :base-directory , (concat org-directory "blog")
   :base-extension "org"
   :publishing-directory , (concat org-directory "jekyll")
   :recursive t
   :publishing-function org-gfm-publish-to-markdown
   :with-toc nil
   :headline-levels 4
   :auto-preamble nil
   :auto-sitemap nil
   :html-extension "html"
   :body-only t)
  ("blog" :components ("gitpages"))))
```

这部分定义了 project 架构。

后面就是一些自动化的处理，主要需要符合 jekyll blog 的输出结构，例如文件名需要带上日期之类。我们需要做的是在 blog 的 \_posts 目录中建立和完成 org 文件，这样在发布时就会自动发布到 jekyll 的 \_posts 中。下面的一些函数主要来自前文的那个链接。

```emacs-lisp
(defvar jekyll-directory (expand-file-name (concat org-directory "blog/"))
  "Path to Jekyll blog.")
;(defvar jekyll-drafts-dir "_drafts/"
;  "Relative path to drafts directory.")
(defvar jekyll-posts-dir "_posts/"
  "Relative path to posts directory.")
(defvar jekyll-post-ext ".org"
  "File extension of Jekyll posts.")
(defvar jekyll-post-template
  "#+BEGIN_EXPORT html\n---\nlayout: post\ntitle: %s\nexcerpt: \ncategories:\n  -  \ntags:\n  -  \n---\n#+END_EXPORT\n\n* "
  "Default template for Jekyll posts. %s will be replace by the post title.")

(defun jekyll-make-slug (s)
  "Turn a string into a slug."
  (replace-regexp-in-string
   " " "-" (downcase
            (replace-regexp-in-string
             "[^A-Za-z0-9 ]" "" s))))

(defun jekyll-yaml-escape (s)
  "Escape a string for YAML."
  (if (or (string-match ":" s)
          (string-match "\"" s))
      (concat "\"" (replace-regexp-in-string "\"" "\\\\\"" s) "\"")
    s))

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

使用起来也非常简单，直接 `M-x my-pages-start-post` 来开始一篇文章，输入标题，然后就进入 org mode 写文章。写好后，直接 `M-x org-publish-current-project` 或使用 `M-x org-publish-project` 并选择 blog 项目即可。这样就完成了本地的 jekyll 文件，然后将 jekyll 整个工程推送到 gitpages 即可。
