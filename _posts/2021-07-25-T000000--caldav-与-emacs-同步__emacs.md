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

CalDav即远程日程信息访问（共享）协议，具体见 RFC4791，是一套远程日历同步的协议。其日历用的是 iCalendar 格式，这个格式的后缀是 .ics，其中包括了很多日历常用的字段，支持的相当多的邮箱系统，包括苹果。这个协议建立在 WebDav 之上，WebDav 常常用于网盘的同步，例如坚果云以及 DropBox等。CalDav可以当作是专门用于日历的 WebDav。与之类似的还有CardDav，主要用于同步通讯录。目前苹果，谷歌，包括国内的钉钉，飞书等日历都支持该协议。简单来说，使用 CalDav 协议可以实现各种日历的同步与日程的提醒。


# 钉钉日历

工作中使用钉钉比较多，其日历还算比较好用，需要预约开会或者安排事情之类的，可以直接给人发过去一个邀请，接受之后大家都会互相知道各自的空闲时间与安排。但是钉钉日历有个比较奇葩的地方，就是默认只能从其他的日历导入，例如手机自带的日历上的日程，但是不支持导出到其他日历。说起来钉钉是一个和工作强相关的软件，如果将自己的日历同步给他总有些奇怪，所以最好还是能将其日程自动导出到系统日历进行管理与同步。这样工作相关的可以只在钉钉上处理，其他的在系统或其他软件处理。在钉钉日历的选项中转了一圈之后惊讶的发现，这货竟然支持 CalDav，只不过默认没开启，教程也不明显。而各种形形色色的日历软件，不支持 CalDav 的也很多，所以一直没发现。在钉钉日历的选项里面直接打开 CalDav，会生成一个用户名和密码，以及一个用于同步的 url。

获得这些信息，就可以在支持 CalDav 的日历系统里面添加了，默认的 iOS 日历，以及 macOS 系统自带的日历都支持这个协议，添加一下，马上就能把日历同步过来。据说各大 Android 原生系统还不支持，我看了一下，比较意外的是小米手机自带的日历也支持 CalDav。对于其他的安卓手机，可以在 fdroid 中下载 DAVx 这个软件，就可以将所有支持 CalDav 的日历导入到系统自带日历了。这里的设置就不详细说了，下面专门说下 Emacs.


# Org Mode

Emacs 的 Org mode 是当前最好的纯文本日程管理软件，随着 Org Roam 包的推出，又借着 Roam Research 的概念火了一把。Org Roam 简单来说就是一套支持引用，互相链接的 org 文件以及系统。在 Org mode 中，通过 org-agenda，以及各种时间戳来管理日程，结合各种 notification 和 clock-in/out 方式，能很好的实现任务的跟踪和记录。特别适合类似《奇特的一生》中介绍的的时间管理方式。也有非常多的文章说如何在 org mode 里面实现 GTD。下面主要看如何通过 CalDav 来同步钉钉的日程。

首先我们找到的是 [Org-CalDav](https://github.com/dengste/org-caldav) 这个包，可以通过 org mode 来双向与支持 caldav 的日历同步，其配置也很简单。

```emacs-lisp
(add-to-list 'load-path "/path/to/org-caldav")

(require 'org-caldav)

(defconst cal-dav-user-name "<username>")
(setq org-caldav-url (concat "https://calendar.dingtalk.com/dav/" cal-dav-user-name))
(setq org-caldav-calendar-id "primary")
```

上面那几个参数设置也比较曲折，后来读了源码里才搞明白。 `org-caldav` 会把 `org-caldav-url` 和 `org-caldav-calendar-id` 一起组装成日历的真实地址，然后去请求。对于钉钉来说，按照上述配置后，请求的 `org-caldav-url` 地址是 `https://calendar.dingtalk.com/dav/<username>` 。配置完成，执行 `M-x org-caldav-sync` ，发现倒是没什么报错，但是什么都没有，却提示同步完毕，通过 curl 去请求，发现其实有返回，返回也正正是日历信息。

```bash
curl -D - -X GET --basic -u mylogin:mypassword URL
```

这里 `URL` 表示对应的日历地址，返回值有日历内容，但是在 emacs 中无法同步下来。跟踪源码发现， `org-caldav` 默认日历是 `.ics` 结尾，并且对此进行了过滤，而钉钉里面日历没有后缀，所以就没有显示了。于是设置

```emacs-lisp
(setq org-caldav-uuid-extension "")
```

再 `M-x org-caldav-sync` 验证，就能看到相应的日历被同步到 `org-caldav-inbox` 文件中了。再做一些简单的定制，就可以使用了。

当然还有一些小问题，例如 utf-8 格式的中文字串显示不正确、以及默认不支持 repeated 的任务，我做了一点小修改，放到 [org-caldav](https://github.com/smallzhan/org-caldav) 里面了，最后的配置如下：

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

将 inbox 文件加入到 `agenda-files` 是为了在 `org-agenda` 看到对应的日程。本地对这个文件的修改也可以双向同步到钉钉日历。
