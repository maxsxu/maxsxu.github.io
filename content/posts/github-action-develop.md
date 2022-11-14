+++
draft = true
date = 2022-05-29T22:16:13+08:00
title = "GitHub Action 开发"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
+++


# Caveats
- A step cannot have both the `uses` and `run` keys. 因为 Action 里已经定义了 `run` 和 `shell`
