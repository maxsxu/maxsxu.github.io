+++ 
draft = false
date = 2020-07-04T18:23:42+08:00
title = "macOS Catlina 对 CA 证书的新要求"
description = ""
slug = "" 
tags = ["macOS", "TLS"]
categories = ["Security"]
externalLink = ""
series = []
+++

最近遇到个问题，自签的 CA 证书被 Safari 提示有风险，虽然已导入到 Keychain 且设置信任。此事之前从未出现过，而且在其它 OS 测试也没有问题，所以基本判定是 macOS 系统的问题。找到的官方说明如下:

> **iOS 13 和 macOS 10.15 中 TLS 证书的新安全要求**
> 
> - TLS 服务器证书和 CA 使用的 RSA 密钥长度必须大于或等于 `2048` 位。使用 RSA 密钥长度小于 2048 位的证书不再受信
> - TLS 服务器证书和 CA 必须在签名算法中使用 `SHA-2` 系列的哈希算法。SHA-1 签名的证书不再受信
> - TLS 服务器证书必须在证书的 `SAN` (Subject Alternative Name) 扩展属性中显示服务器的 DNS 名称。证书的 CommonName 中的 DNS 名称不再受信
> 
> 此外，2019 年 7 月 1 日之后颁发的所有 TLS 服务器证书（以证书的 NotBefore 属性为准）必须遵循以下准则：
> 
> - TLS 服务器证书必须包含 `ExtendedKeyUsage（EKU）` 扩展，其值包含id-kp-serverAuth OID
> - TLS 服务器证书的有效期必须为 `825` 天或更短（如证书的 NotBefore 和 NotAfter 字段所示）
> 
> 违反这些新要求的 TLS 服务器连接将失败，并可能导致网络故障，应用程序失败以及网站无法加载到 iOS 13 和 macOS 10.15 的 Safari 中。


特别要注意的是最后一条，证书的有效期不超过 825 天，有效期太长的证书 Chrome 浏览器现在也会提示为安全问题，Chrome 浏览器貌似是要求 CA 的有效期不超过 5 年。如用 `cfssl` 签证书的时候 `expires` 值要小于 19800h，基本没问题。

# 参考
- Requirements for trusted certificates in iOS 13 and macOS 10.15, https://support.apple.com/en-in/HT210176