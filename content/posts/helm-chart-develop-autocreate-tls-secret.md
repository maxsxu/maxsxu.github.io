+++
draft = false
date = 2019-05-23T21:21:24+08:00
title = "Helm Chart 开发: 自动生成 Kubernetes Dashbord 的 TLS Secret"
description = "Helm 部署 Kubernetes Dashbord 自动生成 TLS Secret"
slug = ""
tags = ["Kubernetes", "Helm"]
categories = ["CloudNative"]
externalLink = ""
series = []
+++


# 背景
部署 Kubernetes Dashbord 有很多方式，其中最方便的还是通过 Helm 来部署。
官方提供了 [Helm Chart](https://github.com/helm/charts/stable/kubernetes-dashboard)，在其 README 中说只需要执行下面一条命令就可以部署 Dashboard:

```bash
helm install stable/kubernetes-dashboard
```

但实际上发现部署失败，因为缺少 TLS Secret。通过查看 Chart 源码，可以发现 `values.yaml` 文件中有如下定义:

```YAML
  ## Kubernetes Dashboard Ingress TLS configuration
  ## Secrets must be manually created in the namespace
  ##
  # tls:
  #   - secretName: kubernetes-dashboard-tls
  #     hosts:
  #       - kubernetes-dashboard.domain.com
```

原来还需要先手动创建 Secret 才行，这也太不优美了，于是产生了增强 Kubernetes Dashboard 的 Chart 的想法。


# 目标
支持自动生成证书和 Secret，能够真正达到使用 Helm 一行命令成功部署 Kubernetes Dashboard，不需要额外操作。

# 解决方案
首先梳理了下整个 Chart 的执行逻辑。

## 分析
官方 Chart 的目录结构如下:

```bash
➜  kubernetes-dashboard
├── templates
│   ├── clusterrole-readonly.yaml
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── pdb.yaml
│   ├── rolebinding.yaml
│   ├── role.yaml
│   ├── secret.yaml
│   ├── serviceaccount.yaml
│   └── svc.yaml
├── Chart.yaml
├── OWNERS
├── README.md
└── values.yaml
```

其中 `Chart.yaml` 描述 Chart 的元信息，`values.yaml` 文件描述 Chart 默认配置，`templates` 目录包含提供给 Kubernetes 的各种资源模板 YAML 文件。

我的基本思路: 在 Chart 中自动生成 TLS 证书和 Secret，然后在 Ingress 中挂载自动生成的 Secret 即可。
下面描述一下这两个核心步骤: 自动生成 `Secret` 和 Ingress 挂载 `Secret`。

## 第一步: 自动生成 `Secret`
为了能够自动生成 Secret，需要借助 [sprig](https://github.com/Masterminds/sprig) 的一些函数。[sprig](https://github.com/Masterminds/sprig) 是一个非常强大的 Go 模板库，在许多项目中大量使用，包括 Helm ，可以分析 Helm 源码看到。

`sprig` 库有与加密和安全相关的函数，其中 `genCA` 和 `genSignedCert` 函数就可以用来生成证书。

`genCA` 函数用于生成一个自签名的 x509 CA 证书，语法如下:
```go
$ca := genCA [CN] [duration]
```

**输入:** 两个参数: `CN` 表示证书持有者的通用名称, `duration` 表示证书的有效天数。

**输出:** 返回一个包含两个属性的证书对象，可将 `$a` 对象传递给 `genSignedCert` 函数以使用此 CA 对证书进行签名。

- `Cert` 属性表示 PEM 编码的证书
- `Key` 属性PEM编码的私钥

`genSignedCert` 函数用于生成一个由指定 CA 签名的 x509 证书，语法如下:
```go
$ca := genCA "foo-ca" 365
$cert := genSignedCert "foo.com" (list "10.0.0.1" "10.0.0.2") (list "bar.com" "bat.com") 365 $ca
```

**输入:** 有五个参数:

- 证书持有者的通用名称
- IP 的可选列表，可以为 `nil`
- 备用 DNS 名称的可选列表，可以为 `nil`
- 证书的有效天数
- CA 证书，由 `genCA` 生成

**输出:** 与 `genCA` 一样，返回一个包含 `Cert` 和 `Key` 属性的证书对象。

了解上面两个函数的用法后，在 `secret.yaml` 模板文件定义自动生成证书:

```YAML
{{- $ca := genCA "kubernetes-dashboard-ca" 365 }}
{{- $cert := genSignedCert .Values.ingress.tls.hosts.name nil (list .Values.ingress.tls.hosts.name) 365 $ca }}
```

为了便于取值，其中 `.Values.ingress.tls` 变量已在 `values.yaml` 文件中重新定义为字典 (`dict`) 而不是官方的列表 (`list`)。这样获得了自动生成的CA和证书后，我们就可以继续定义自动生成的 Secret 了:

```YAML
apiVersion: v1
kind: Secret
metadata:
  name: "{{ template "kubernetes-dashboard.tlsSecretName" . }}"
  labels:
    app: {{ template "kubernetes-dashboard.name" . }}
    chart: {{ template "kubernetes-dashboard.chart" . }}
    heritage: {{ .Release.Service }}
    release: {{ .Release.Name }}
type: kubernetes.io/tls
data:
  tls.crt: {{ $cert.Cert | b64enc | quote }}
  tls.key: {{ $cert.Key | b64enc | quote }}
  ca.crt: {{ $ca.Cert | b64enc | quote }}
{{- end }}
{{- end }}
```

其中 `kubernetes-dashboard.tlsSecretName` 是自动生成的 Secret 名称，是我在 `_helpers.tpl` 模板辅助文件中定义的模板变量。

## 第二步: `Ingress` 挂载 `Secret`
`ingress.yaml` 模板文件描述了 Ingress 资源的定义，可以看到官方的做法是直接填充 TLS 配置值列表。配置值来源于提前生成的 Secret。

```YAML
  {{- if .Values.ingress.tls }}
  tls:
{{ toYaml .Values.ingress.tls | indent 4 }}
  {{- end -}}
```

现在有了自动生成的 Secret 资源，可以直接挂载了:

```YAML
{{- $tls := .Values.ingress.tls -}}

{{- if $tls }}
  tls:
    - hosts:
        - {{ $tls.hosts.name }}
      {{- if $tls.secretName }}
      secretName: {{ $tls.secretName }}
      {{- else }}
      secretName:  "{{ template "kubernetes-dashboard.tlsSecretName" . }}"
      {{- end }}
{{- end }}
```


# 总结
通过上面的 Chart 增强操作之后，就可以实现一键部署 Dashboard 了，不需要提前手动创建 Secret 了（当然，也支持手动创建的）。

```bash
helm install --name kubernetes-dashboard --namespace kube-system ./kubernetes-dashboard
```

不过目前的做法还是存在一些缺点: 一是不支持多个 TLS Secret，因为目前为了取值方便使用 `dict` 类型的 tls 值，后面会改进支持 `list` 类型；二是重新部署或者升级又会生成新的证书和 Secret，对用户不友好。


# 参考
- sprig | Cryptographic and Security Functions, http://masterminds.github.io/sprig/crypto.html
