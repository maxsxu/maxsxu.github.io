+++
draft = false
date = 2022-05-04T18:13:49+08:00
title = "Kubernetes 1.24 发布"
description = ""
slug = ""
tags = ["Kubernetes"]
categories = ["CloudNative"]
externalLink = ""
series = []
+++

Kubernetes 1.24 于 2022.05.03正式发布。同时这也是 2022 年的第一个版本。

此版本包含 46 个增强功能：14 个增强功能已升级为稳定版，15 个增强功能正在进入 beta 阶段，13 个增强功能正在进入 alpha 阶段。此外，两个功能已被弃用，两个功能已被删除。

主要包含两个重大变更：

- `Dockershim` 被彻底移除
- Beta API 默认关闭

## Dockershim 从 kubelet 中移除
在 v1.20 中弃用后，`dockershim` 组件已从 Kubernetes v1.24 中的 `kubelet` 中删除。从 v1.24 开始，用户将需要使用其他受支持的运行时之一（例如 `containerd` 或 `CRI-O`），或者如果您依赖 Docker Engine 作为容器运行时，则使用 `cri-dockerd`。

## Beta API 默认关闭
默认情况下，不会在集群中启用新的 beta API。默认情况下，现有 beta API 和现有 beta API 的新版本将继续启用。


# References
- Kubernetes 1.24: Stargazer, https://kubernetes.io/blog/2022/05/03/kubernetes-1-24-release-announcement/
