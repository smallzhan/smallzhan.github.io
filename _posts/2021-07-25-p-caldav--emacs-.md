---
layout: post
title: CalDav 与 Emacs 同步
comments: true
excerpt: 
categories:
  - emacs  
tags:
  - emacs 
---


# CalDav 是什么

CalDav即远程日程信息访问（共享）协议，具体见 RFC4791。 是一套远程日历同步的协议，其日历用的是 iCalendar 格式，这个格式的后缀是 .ics，其中包括了很多日历常用的字段，支持的邮箱非常多，苹果也默认支持。这个协议建立在 WebDav 之上，WebDav 部分网盘支持，例如坚果云以及常见的 DropBox。当然 WebDav 主要用在网盘领域，是同步文件的好手。CalDav可以当作是专门用于日历的 WebDav。与之类似的还有CardDav，主要用于同步通讯录的。目前苹果，谷歌等日历都采用该协议。简单来说，用 CalDav 可以实现各种日程提醒，日历软件的同步。


# 钉钉日历

工作中使用钉钉比较多，特别是其日历比较好用，需要预约开会或者安排事情之类的，可以直接给人发过去一个邀请，接受之后大家都知道互相的空间时间与安排。但是钉钉日历有个比较奇葩的地方，就是默认，他可以导入其他的日历，例如手机自带的日历上的日程，或其他等，但是不支持导出到其他日历。因为说起来钉钉是一个和工作强相关的软件，如果将自己的日历同步给他总有些怪怪的，最好还是能将其日程专门导入到一个固定的地方管理与同步。这样工作相关的可以在钉钉上处理，其他的在其他软件处理。在钉钉日历的选项中转了一圈之后惊讶的发现，这货竟然支持 CalDav，只不过默认没打开也没教程，而各种形形色色的日历软件，不支持 CalDav 的也很多，所以一直没发现。在钉钉日历的选项里面直接打开 CalDav，会生成一个用户名和密码，以及一个用于同步的 url。

获得用户名，密码和 url 之后，就可以在支持 caldav 的日历系统里面添加了，默认的 iOS 日历，以及 macOS 系统自带的日历都支持这个协议，添加一下，马上就能把日历同步过来。据说各大 Android 目前还不支持，但是我看了一下，比较意外的是小米手机自带的日历也支持这个 CalDav。这里的设置就不详细说了，下面专门说下 Emacs.


# Org Mode

Emacs 的 Org mode 是当前最好的纯文本日程管理软件，当然随着发展，目前由于 Org Roam 的加入，使得其又借着 Roam Research 的概念火了一把。那个其实是另一个故事了，支持引用，互相链接的 org 文件。在 org mode 中，通过 agenda，以及各种时间戳来管理日程，并且结合各种 notification 方式，能很好的实现任务的跟踪和记录。特别适合类似《奇特的一生》这样的管理方式。当然也有非常多的文章说如何在 org mode 里面实现 GTD。这里也不多说，主要看如何通过 caldav 来同步钉钉的日程。

首先我们找到的是 [Org-CalDav](https://github.com/dengste/org-caldav) 这个包，号称是可以通过 org mode 来双向与支持 caldav 的日历同步，其配置也很简单。

```emacs-lisp
(add-to-list 'load-path "/path/to/org-caldav")

(require 'org-caldav)

(defconst cal-dav-user-name "<username>")
(setq org-caldav-url (concat "https://calendar.dingtalk.com/dav/" cal-dav-user-name))
(setq org-caldav-calendar-id "primary")
```

上面那几个参数也比较曲折，后来还是去源码里面搞清楚的， `org-caldav` 会把 `org-caldav-url` 和 `org-caldav-calendar-id` 一起组装起来，组成这个日历的真实地址，然后去访问，而对于钉钉来讲，这个地址就是 `https://calendar.dingtalk.com/dav/<username>` 所以这里按照上面那么写。写好之后，直接执行 `M-x org-caldav-sync` ，发现倒是没什么报错，但是什么都没有，提示同步完毕，而这个时候通过 curl 去检查会发现其实是有东西的。

```bash
curl -D - -X GET --basic -u mylogin:mypassword URL
```

这里把 URL 写成组装的日历地址，发现，其实是有日历内容，但是同步不下来。所以跟踪进去代码发现，原来 `org-caldav` 默认日历都是 `.ics` 结尾，但是钉钉里面的日历没有这个后缀名，因此全部过滤了，也好办

```emacs-lisp
(setq org-caldav-uuid-extension "")
```

到这里基本就好了，再 `org-caldav-sync` 就能看到相应的日历下来，放在 `org-caldav-inbox` 这个文件里面了，再做一些简单的定制，就没问题了。

当然还有一些小的问题，例如 utf-8 格式的中文字串显示之类的，以及默认不支持 repeated 的任务，我做了一点小修改，放到 [org-caldav](https://github.com/smallzhan/org-caldav) 里面了，最后的配置如下：

```emacs-lisp
(use-package org-caldav
  :after org
  :load-path "/path/to/org-caldav"
  :init
  (setq org-caldav-url "https://calendar.dingtalk.com/dav/<username>")
  (setq org-caldav-calendar-id "primary")
  (setq org-caldav-uuid-extension "")
  (setq org-caldav-sync-direction 'twoway)
  (setq org-caldav-inbox (concat org-directory "agenda/dingtalk.org"))
  (setq org-caldav-files `(,org-caldav-inbox))
  (add-to-list 'org-agenda-files org-caldav-inbox)
  :config
  (require 'org-caldav))
```

加入到 `agenda-files` 之后就可以直接用 `org-agenda` 看到对应的日程了。本地对这个文件修改的也可以双向同步到钉钉日历。
