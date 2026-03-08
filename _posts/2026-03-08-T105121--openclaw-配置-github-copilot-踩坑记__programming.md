---
layout: post
title: openclaw 配置 github-copilot 踩坑记
comments: t
date: 2026-03-08
excerpt:
categories:
  - programming
tags:
  - programming
---

一直有 github-copilot 的订阅，在其他订阅方式中，这个其实是个比较有意思的，他集成了御三家 (OpenAI, Anthropic, Google) 的主流模型，可以深度与 vscode 天生集成，并且也提供了 copilot 的命令行，而且在 opencode 中原生有支持。定价方面也还好，10$/month 的 Pro 套餐，如果不是使用强度很高，也没啥问题。35 刀每月的量比这个多了 5 倍，基本上也可以略高强度使用了。

后来开始玩 openclaw，之前官方并不支持 copilot 订阅，我试了试 deepseek 之类的，一会会用了非常多的 token，就一直没怎么正经用。到处找一些免费的模型之类的使用，找到 nvidia 的模型，配是配上了，但是响应特别慢，说话要等一分钟可能才有回应，忍不了了，也想试试好的模型是什么感觉。想到正好也支持了 copilot，就开始配上了。

按照官方文档[这里](https://docs.openclaw.ai/zh-CN/providers/github-copilot#github-copilot)，一开始很顺利。熟悉的味道，直接 github 做个 device 认证，就来了。设置模型时，按照一句：

> openclaw models set github-copilot/gpt-4o

也设置成功了，这时候再去找龙虾，响应飞快。就是这个龙虾不太聪明，很多工具不了解也不会用，随便问下其他的事情等等的，训练内的没啥问题，需要调用工具的，能力上就差多了。所以这个龙虾除了聊天基本上是个残废。不行，那测别的模型吧。

马上试试 set 一个 gpt-5.4 来用用，刚从 vscode 里面看似乎有了。

> openclaw models set github-copilot/gpt-5.4

测试，说找不到模型，那可能是 pro 还没提供吧。设置 claude 试试，这个确认有，结果一调用，反馈 400，有点懵。明明用的好好的。我在其他地方找到了模型的名字，直接去改配置文件，因为文档里面配置文件有一句：

> { agents: { defaults: { model: { primary: "github-copilot/gpt-4o" } } }, }

结果我改成 gemini 或者 claude，或者 gpt-5.3 之类的高级模型都不行。

到目前为止，订阅里面只有 gpt-4o 能用，其他都不行。我就跟龙虾聊，让他去试试 github-copilot 有哪些模型，分别调用试试。他去找了源码，然后告诉我，有哪些哪些模型，我就让去试，结果试下来告诉我，除了 4o，gpt-5.4 是没有，其他都不行。然后告诉我 openclaw 有个白名单，会决定哪些模型能调用，哪些不能。我说那你去改，他去改了一个叫做 "models.json" 的文件，结果每次我去看，这个文件里面的模型都是空，再问，他说确定改了，可能是系统覆盖了。

有点僵局了，这时候我突然看到有些模型提示是 model not allowed，有些提示似乎是调用了，似乎有些是被 openclaw 拦住的，有些是被 api 拦住的。所以我就让龙虾自己去调用，同时监控 log，他告诉我，claude 这些，调用到 api 了，结果是 api 拦住的，而 gemini 这些，直接调用就不允许了。我突然想起来，配置文件 openclaw.json 中，关于 models 里面，claude, gpt-4o 这些都是写了的。而其他的没写，于是找他要了模型列表，手动在 models 里面添加了。再让他测，除了 claude 之外，其他都成功了。

那 claude 是为什么？突然想起来，之前 anthropic 说不给中国服务，因此如果 ip 在国内，copilot 或 vscode 里面都是看不见 claude 类模型的。于是去调整了下梯子配置，再试 claude，立刻就成功了。

因为 copilot 中 gpt-4o 和 gpt-5-mini 是 0x 的，相当于是免费使用，因此先用 gpt-5-mini 试了试，结果创建定时任务，墨迹半天，一会说没权限，一会说创建不了。看来他完全不知道 openclaw 的接口啊，把文档喂给他，能创建了，结果不能调用，其中格式有点乱，还乱找问题，死循环了。而且这个模型还特别啰嗦，每次都给你提供1234 好多方案，然后获得回复才开始。于是将主模型换成 claude，同样一个任务，过一会，回复说搞定了。

突然想到，如果一开始没有网络问题，直接就配置好了 claude, 可能好多问题一早就解决了，不会整这么久。所以龙虾最大的坑在于，一定要给他好模型，不要贪图便宜，过去的模型，当时可能非常厉害，在现在的迭代强度下，调用工具的能力，解决问题的能力，其差距大概是一个高材生对上村头 2 傻子类似的体验。
