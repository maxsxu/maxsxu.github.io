+++ 
draft = false
date = 2019-09-21T16:54:44+08:00
title = "阿里云开发者社区云原生应用大赛"
description = "阿里云开发者社区云原生应用大赛, kubernetes-dashborad-tls, helm chart"
slug = "" 
tags = ["Kubernetes", "Helm"]
categories = ["Contest"]
externalLink = ""
series = []
+++

前一阵子看到钉钉群里有同学分享了这个比赛，点进去看了下，结果第二天就要截止了。。。但是觉得大赛出发点非常好（主要是又有机会拿 T 恤，哈哈哈），恰好上次写了一个可以自动生成 TLS Secret 的 kubernetes-dashbord Helm Chart，于是连夜提交了一波，项目地址为: [kubernetes-dashborad-tls](https://github.com/cloudnativeapp/charts/tree/master/submitted/kubernetes-dashboard-tls)

大赛要求必须使用 `Helm v3`，就是要这种前瞻性的比赛项目，有利于优秀技术的推广。关于 `helm v3` 相对于 `helm v2` 又很多重大的变化，当然最突出的改变在于去掉了 `Tiller` 这个组件。

# 大赛介绍
引用[官方介绍](https://developer.aliyun.com/special/apphubchallenge):

> 云原生应用，是指符合“云原生”理念的应用开发与交付模式，这是当前在云时代最受欢迎的应用开发最佳实践。

> 在现今的云原生生态当中，已经有很多成熟的开源软件被制作成了 Helm Charts，使得用户可以非常方便地下载和使用，比如 Nginx，Apache、Elasticsearch、Redis 等等。不过，在开放[云原生应用中心 App hub（Helm Charts 中国站)](https://developer.aliyun.com/hub) 发布之前，国内用户一直都很难直接下载使用这些 Charts。而现在，AppHub 不仅为国内用户实时同步了官方 Helm Hub 里的所有应用，还自动替换了这些 Charts 里所有不可访问的镜像 URL（比如 `gcr.io`, `quay.io` 等），终于使得国内开发者通过 helm install “一键安装”应用成为了可能。

> 而云原生应用开发大赛，旨在鼓励和普及 Helm Charts 在国内的使用，帮助国内开发者通过云原生的方式打包和分发自己的应用，从而更好的借助云原生的浪潮，让自己的软件在云时代发挥出最大的能量！

# 竟然获奖了!

一件阿里云 T 恤

![](/images/tshirt-aliyun.jpeg)


