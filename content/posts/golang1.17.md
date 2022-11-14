+++
draft = true 
date = 2021-08-17T13:41:24+08:00
title = "Golang 1.17 发布"
description = ""
slug = ""
tags = ["Golang"]
categories = ["Programing Language"]
externalLink = ""
series = []
+++

## Golang module 依赖图修剪
相比之前的版本，直接依赖和间接依赖默认都放在 `go.mod` 中的同一个 `require` 块中。
go1.17 将直接依赖和间接依赖分别放在 `go.mod` 中两个不同的 `require` 块中。


# References
- Module graph pruning, https://go.dev/ref/mod#graph-pruning
