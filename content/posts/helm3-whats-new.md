+++ 
draft = false
date = 2019-11-13T17:01:21+08:00
title = "Helm 3 的新特性"
description = ""
slug = "" 
tags = ["Helm"]
categories = ["CloudNative"]
externalLink = ""
series = []
+++

本文介绍 Helm 3 中引入的一些重大更改。

# 重大改变
## 移除了 Tiller
在 Helm 2 的开发周期中，我们引入了 Tiller。Tiller 在团队协作中共享集群时扮演了重要角色。 它使得不同的操作员与相同的版本进行交互称为了可能。

Kubernetes 1.6 默认使用了基于角色的访问控制（RBAC），在生产环境对 Tiller 的锁定使用变得难于管理。 由于大量可能的安全策略，我们的立场是提供一个自由的默认配置。这样可以允许新手用户可以乐于尝试 Helm 和 Kubernetes 而不需要深挖安全控制。 不幸的是这种自由的配置会授予用户他们不该有的权限。DevOps 和 SRE 在安装多用户集群时不得不去学习额外的操作步骤。

在听取了社区成员在特定场景使用 Helm 之后，我们发现 Tiller 的版本管理系统不需要依赖于集群内部用户去维护 状态或者作为一个 Helm 版本信息的中心 hub。取而代之的是，我们可以简单地从 Kubernetes API server 获取信息， 在 Chart 客户端处理并在 Kubernetes 中存储安装记录。

Tiller 的首要目标可以在没有 Tiller 的情况下实现，因此针对于 Helm 3 我们做的首要决定之一就是完全移除 Tiller。

随着 Tiller 的消失，Helm 的安全模块从根本上被简化。Helm 3 现在支持所有 Kubernetes 流行的安全、 身份和授权特性。Helm 的权限通过你的 kubeconfig 文件进行评估。 集群管理员可以限制用户权限，只要他们觉得合适， 无论什么粒度都可以做到。版本发布记录和 Helm 的剩余保留功能仍然会被记录在集群中。

## 移除了 `helm serve`
`helm serve` 命令可以在你本地机器运行一个 Chart 仓库用于开发目的。 然而作为一个开发工具并没有受到太多利用，并且设计上有很多问题。最终我们决定移除它， 拆分成了一个插件。

对于 helm serve 的类似经历，可以查看本地文件系统存储选项在 ChartMuseum 和 servecm plugin.

## CLI 命令重新命名
为了更好地从包管理器中调整不当措辞，`helm delete` 被重命名为 `helm uninstall`。 `helm delete` 依然作为 `helm uninstall` 的别名保留， 因此其他格式也能使用。

Helm 2 中为了清除版本清单，必须提供 `--purge` 参数。这个功能现在是默认使用的。 为保留之前的操作行为，要使用 `helm uninstall --keep-history`。

另外，其他一些重命名的命令提供了以下约定：

- `helm inspect` -> `helm show`
- `helm fetch` -> `helm pull`

这些命令都保留了老的动词作为别名，因此您能够使用任意一种格式。

## 发布名称现在限制在 namespace 范围内
随着 Tiller 的移除， 每个版本的信息需要保存在某个地方。 在 Helm 2 中，是存储在 Tiller 相同的命名空间中。 实际上这意味着一个发布版本使用一个名称，其他发布不能使用相同的名称， 即使在不同的命名空间中也不行。

在 Helm 3 中，特定的版本信息作为发布本身存储在相同的命名空间中。 意味着用户现在可以在两个分开的命名空间中使用 `helm install wordpress stable/wordpress`， 并且每个都能使用 `helm list` 改变当前命名空间。 (例如 `helm list --namespace foo`)。

与本地集群命名空间更好的一致性，使得 `helm list` 命令不再需要默认列出所有发布版本的列表。 取而代之的是，仅仅会在命名空间中列出当前 kubernetes 上下文的版本。 (也就是说运行 `kubectl config view --minify` 时会显示命名空间). 也就意味着您在执行 `helm list` 时必须提供 `--all-namespaces` 标识才能获得和 Helm 2 同样的结果。

## 更改了 Go 的 path 导入

在 Helm 3 中，Helm 将 Go 的 import 路径从 `k8s.io/helm` 切换到了 `helm.sh/helm/v3`。如果你打算 升级到 Helm 3 的 Go 客户端库，确保你已经更改了 import 路径。

## Capabilities

`.Capabilities` 内置对象会在已经简化的渲染阶段生效。

## 使用 JSON 格式验证 Chart Values

chart values 现在可以使用 JSON 结构了。这保证用户提供 value 可以按照 chart 维护人员设置的结构排列， 并且当用户提供了错误的 chart value 时会有更好错误提示。

当调用以下命令时会进行 JSON 格式验证：

- `helm install`
- `helm upgrade`
- `helm template`
- `helm lint`

## 将 `requirements.yaml` 合并到了 `Chart.yaml`
Chart 依赖体系从 requirements.yaml 和 requirements.lock 移动到 Chart.yaml 和 Chart.lock。我们推荐在 Helm 3 的新 chart 中使用新格式。不过 Helm 3 依然可以识别 Chart API 版本 1 (v1) 并且会加载已有的 requirements.yaml 文件。

Helm 2 中，requirements.yaml 看起来是这样的:

```
dependencies:
- name: mariadb
  version: 5.x.x
  repository: https://charts.helm.sh/stable
  condition: mariadb.enabled
  tags:
    - database
```

Helm 3 中， 依赖使用了同样的表达方式，现在 Chart.yaml 是这样的：

```
dependencies:
- name: mariadb
  version: 5.x.x
  repository: https://charts.helm.sh/stable
  condition: mariadb.enabled
  tags:
    - database
```

Chart 会依然下载和放置在 charts/ 目录， 因此 charts/ 目录中的子 chart 不作修改即可继续工作。


## Name (或者 `--generate-name`) 安装时是必需的
Helm 2 中，如果没有提供名称， 会自动生成一个名称。在生产环境，这被证明是一个麻烦事而不是一个有用的特性。 而在 Helm 3 中，如果 `helm install` 没有提供 name，会抛异常。

如果仍然需要一个自动生成的名称，您可以使用 `--generate-name` 创建。

## 推送 Chart 到 OCI 注册中心
这是一个 Helm 3 中的实验性特性。使用时需要设置环境变量 `HELM_EXPERIMENTAL_OCI=1`。

Chart 仓库在较高层次上是一个存储和分发 Chart 的地址。Helm 客户端打包并将 Chart 推送到 Chart 仓库中。 简单来说，Chart 仓库就是一个基本的 HTTP 服务器用来存放 index.yaml 文件和打包的 chart。

Chart 仓库 API 满足最基本的需求有一些好处，但是有些缺点开始显现出来：

- Chart 仓库很难在生产环境抽象出大部分的安全性实现。在生产环境有一个认证和授权的标准 API 就显得格外重要。
- Helm Chart 的初始化工具用来签名和验证 chart 的完整性和来源，在 chart 的发布过程中是可选的。
- 在多客户场景中，同一个 chart 可以被其他客户上传，同样的内容会被存储两次。chart 仓库可以更加智能地处理 这个问题，但并不是正式规范的一部分。
- 在安全的多客户实现中使用单一的索引文件进行搜索、元数据信息存放和获取 chart 会变得困难和笨拙。
- Docker 的分发项目（也称作 Docker 注册中心 v2）是 Docker 注册项目的继承者。 很多主要的云供应商都提供项目 分发，很多供应商都提供相同的产品， 这个分发项目得益于多年的强化、安全性实践和对抗测试。

请查看 `helm help chart` 和 `helm help registry` 了解如何打包 chart 并推送到 Docker 注册中心的更多信息。

## Library chart 支持
Helm 3 支持的一类 chart 称为 “library chart”。 这是一个被其他 chart 共享的 chart， 但是它自己不能创建发布组件。library chart 的模板只能声明 define 元素。 全局范围 内的非 define 内容会被简单忽略。这允许用户复用和共享可在多个 chart 中重复使用的代码片段。 避免冗余和保留 chart DRY。

Library chart 在 Chart.yaml 的依赖指令中声明，安装和管理与其他 chart 一致。

```
dependencies:
  - name: mylib
    version: 1.x.x
    repository: quay.io
```

## Chart.yaml api 版本切换
随着对 library chart 的支持以及 requirements.yaml 合并到 Chart.yaml，客户端可以识别 Helm 2 的包格式而不理解 这些新特性。因此我们将 Chart.yaml 的 apiVersion 从 v1 切换到了 v2。

helm create 现在使用使用新格式创建 chart，默认的 apiVersion 也切换到了这里。

客户端希望同时支持两个版本，进而能够检查 Chart.yaml 的 apiVersion 字段去理解如何解析包格式。

## XDG 基本目录支持
XDG 基本目录规范 是一个定义了配置、数据和缓存文件应该存储在文件系统什么位置的可移植标准。

Helm 2 中，Helm 将所有的信息存储在 ~/.helm (被亲切地称为 helm home)，可以通过设置 $HELM_HOME 环境变量修改，或者使用全局参数 --home。

Helm 3 中，Helm 现在遵守 XDG 基本目录规范使用以下环境变量：

- `$XDG_CACHE_HOME`
- `$XDG_CONFIG_HOME`
- `$XDG_DATA_HOME`

Helm 插件仍然使用 $HELM_HOME 作为 $XDG_DATA_HOME 的别名，以便希望将 $HELM_HOME 作为过渡环境的变量来保证向后兼容性。

一些新的环境变量也通过插件环境变量来兼容以下更改：

- `$HELM_PATH_CACHE` 针对缓存路径
- `$HELM_PATH_CONFIG` 针对配置路径
- `$HELM_PATH_DATA` 针对 data 路径

如果 Helm 插件期望支持 Helm 3，建议使用新的环境变量。

## 自动创建 namespace
当用命名空间创建版本时，命名空间不存在，Helm 2 会创建一个命名空间。 Helm 3 中沿用了其他 Kubernetes 工具的形式，如果命名空间不存在，就返回错误。 如果您明确指定 `--create-namespace` 参数，Helm 3 会创建一个命名空间。

## .Chart.ApiVersion 是怎么回事？
Helm 针对缩略语遵循驼峰命名的典型惯例。我们已经在代码的其他位置做了处理，比如 `.Capabilities.APIVersions.Has`。Helm v3 中，我们将 `.Chart.ApiVersion` 更正成了 `.Chart.APIVersion`。


## 改进升级策略：三路策略合并补丁
Helm 2 使用了一种双路策略合并补丁。在升级过程中，会对比最近一次的 chart manifest 和提出的 chart manifest (通过 helm upgrade 提供)。升级会对比两个 chart 的不同来决定哪些更改会应用到 Kubernetes 资源中。 如果更改是集群外带的（比如通过 kubectl edit），则不会被考虑。结果就是资源不会回滚到之前的状态： 因为 Helm 只考虑最后一次应用的 chart manifest 作为它的当前状态，如果 chart 状态没有更改，则资源的活动状态不会更改。

现在 Helm 3 中，我们使用一种三路策略来合并补丁。Helm 在生成一个补丁时会考虑之前老的 manifest 的活动状态。


## 作为默认存储器的密钥
在 Helm 3 中，密钥被作为 默认存储驱动使用。 Helm 2 默认使用 ConfigMaps 记录版本信息。在 Helm 2.7.0 中，新的存储后台使用密钥来存储版本信息， 现在是 Helm 3 的默认设置。

Helm 3 默认允许更改密钥作为额外的安全措施在 Kubernetes 中和密钥加密一起保护 chart。

静态加密密钥 在 Kubernetes 1.7 中作为 alpha 特性可以使用了，在 Kubernetes 1.13 中变成了稳定特性。 这允许用户静态加密 Helm 的发布元数据，同时也是一个类似 Vault 的以后可扩展的良好起点。


# 参考
- Changes since Helm 2, https://helm.sh/docs/faq/changes_since_helm2/