---
layout: post
title: 神奇 AI 找 Bug
comments: t
date: 2026-03-01
excerpt:
categories:
  - programming
tags:
  - programming
---

此前将自己的 Emacs Org 相关函数用 AI 重新实现了一遍，其中有个功能是统计一天的计时情况。以前的函数只能统计当天的，后来借助 AI 将其扩展了一下，做成了可以统计任意一天时间的函数，同时还有看板以及 TODO 的函数，这些函数统一需要展示一个 org buffer 界面，因此就把这些显示封装了一个函数，具体如下。

```emacs-lisp
(defun my/render-view-buffer (buffer-name title refresh-fn body-fn &optional skip-footer)
  "创建并显示一个只读的 org-mode 视图缓冲区。
BUFFER-NAME: 缓冲区名称
TITLE: 标题（包含emoji图标）
REFRESH-FN: 刷新函数符号（用于 'g' 键）
BODY-FN: 无参函数，负责插入缓冲区主体内容
SKIP-FOOTER: 可选，为 t 时跳过自动添加底部提示（由 body-fn 自行处理）"
  (with-current-buffer (get-buffer-create buffer-name)
    (let ((inhibit-read-only t))
      (erase-buffer)
      (org-mode)
      (insert (format "#+TITLE: %s\n" title))
      (insert (format "#+DATE: %s\n\n" (format-time-string "%Y-%m-%d %A")))

      ;; 调用主体内容生成函数
      (funcall body-fn)

      ;; 统一的底部提示（除非明确跳过）
      (unless skip-footer
        (insert (make-string my/view-separator-width ?─) "\n")
        (insert (my/--styled-text "Press 'g' to Refresh | 'q' to Quit" 
                            'font-lock-comment-face)))

      ;; 统一的按键绑定
      (local-set-key (kbd "g") refresh-fn)
      (local-set-key (kbd "q") #'quit-window)

      (goto-char (point-min))
      (setq buffer-read-only t)))
  (pop-to-buffer buffer-name))
```

这个函数会挂两个函数，分别绑定到 g 和 q，作为刷新和退出的函数。使用起来是这样的。

```emacs-lisp
(defun my/org-clock-report (&optional _match)
    (interactive)
    (let ((date (org-read-date)))
      (my/render-view-buffer
       "*Org Timeline Report*"
       (format "Report of [%s]" date)
       ...)))

(defun my/org-kanban-view (&optional _match)
  (interactive)
  (my/render-view-buffer
   "*Kanban*"
   "📊 Kanban View"
   #'my/kanban-view
   (lambda ()
     (my/render-task-sections (my/get-kanban-tasks)))))
```

最近发现一个问题，过一段时间，org-mode 里面就不能正常敲 g 了，会变成 `my/org-clock-report` 的调用，说是绑定到了 `org-mode-map`, 检查了半天没找到哪里绑定的，然后就重启了，一开始是好的，过一会就又绑定了，特别奇怪。实在找不到了，心想，用 AI 试试，反正这几个函数本来就是 AI 写出来的。于是就把 `my/org-clock-report` 发给 AI 看，并告诉他：=org-mode= 中的 `g` 变成了这个函数，无法输入正常的 `g` 了，可能是什么原因。 AI 回复的第一句就把我惊到了：

> 这个问题发生的根源，几乎可以肯定隐藏在你的 my/render-view-buffer 这个函数的实现里。

于是我把 `my/render-view-buffer` 发给 AI，马上他就指出问题了：

> 找到了！问题出在 local-set-key 这个函数的“陷阱”上。
> 
> 为什么 g 会在所有 Org 文件里失效？
> 
> 在 Emacs 中，local-set-key 虽然名字叫“本地”，但它的实现逻辑是：修改当前缓冲区所使用的 Keymap。
> 
> 重点来了：大多数 Major Mode（包括 org-mode）为了节省内存，所有的 Org 缓冲区都共用同一个 org-mode-map。 当你在这个渲染函数里执行 (local-set-key (kbd "g") &#x2026;) 时，你实际上是直接修改了全局的 org-mode-map。结果就是：所有的 Org 文件里的 g 都变成了刷新按钮，导致你无法正常输入字符 g。
> 
> ;; &#x2014; 核心修复：创建一个本地 map 副本，防止污染全局 &#x2014; (use-local-map (copy-keymap org-mode-map))

也就是在做 `local-set-key` 之前，复制一份本地的 `keymap` 即可。加上，果然就好了。

不禁特别感慨，不了解这个知识点，可能找 bug 需要找好久好久，也很难知道如何来 debug 。而 AI 在这些问题上的经验已经远远大于人们了，我们确实生在了一个好时代。
