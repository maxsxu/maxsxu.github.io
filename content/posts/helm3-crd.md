+++
draft = true
date = 2020-04-27T14:38:44+08:00
title = "CRD 在 Helm3 中的使用和限制"
description = ""
slug = ""
tags = ["Helm"]
categories = ["CloudNative"]
externalLink = ""
series = []
+++

Kubernetes 提供了一种机制来声明新类型的 Kubernetes 对象。使用 `CustomResourceDefinitions` (CRD)，Kubernetes 开发人员可以声明自定义资源类型。

在 Helm3 中，CRD 被视为一种特殊的对象。它们在 Chart 的其余部分之前安装，并​​且受到一些限制。

CRD YAML 文件应放在 Chart 内的 `crds/` 目录中。多个 CRD（由 YAML 开始和结束标记分隔）可以放在同一个文件中。 Helm 将尝试将 CRD 目录中的所有文件加载到 Kubernetes 中。

CRD 文件不能被模板化。它们必须是纯 YAML 文档。

当 Helm 安装一个新的 Chart 时，它会上传 CRD，暂停直到 API Server 使 CRD 可用，然后启动模板引擎，渲染 Chart 的其余部分，并将其上传到 Kubernetes。
由于这种排序，CRD 信息在 Helm 模板中的 `.Capabilities` 对象中可用，并且 Helm 模板可以创建在 CRD 中声明的对象的新实例。

# 限制
与 Kubernetes 中的大多数对象不同，CRD 是全局安装的。出于这个原因，Helm 在管理 CRD 时采取了非常谨慎的方法。 CRD 受到以下限制：

- 永远不会重新安装 CRD。如果 Helm 确定 `crds/` 目录中的 CRD 已经存在（无论版本如何），Helm 将不会尝试安装或升级。

- CRD 永远不会在升级或回滚时安装。 Helm 只会在安装操作时创建 CRD。

- CRD 永远不会被删除。删除 CRD 会自动删除集群中所有命名空间中的所有 CRD 内容。因此，Helm 不会删除 CRD。

鼓励想要升级或删除 CRD 的操作员手动执行此操作并非常小心。

# 示例
例如，如果您的 Chart 在 `crds/` 目录中有 CronTab 的 CRD，您可以在 `templates/` 目录中创建 CronTab 类型的实例：

```
crontabs/
  Chart.yaml
  crds/
    crontab.yaml
  templates/
    mycrontab.yaml
```

`crontab.yaml` 文件必须包含没有模板指令的 CRD：

```
kind: CustomResourceDefinition
metadata:
  name: crontabs.stable.example.com
spec:
  group: stable.example.com
  versions:
    - name: v1
      served: true
      storage: true
  scope: Namespaced
  names:
    plural: crontabs
    singular: crontab
    kind: CronTab
```

然后模板 `mycrontab.yaml` 可能会创建一个新的 CronTab（像往常一样使用模板）：

```
apiVersion: stable.example.com
kind: CronTab
metadata:
  name: {{ .Values.name }}
spec:
   # ...
```

Helm 将确保 CronTab 类型已安装并且在 Kubernetes API Server 中可用，然后再继续安装在 `templates/` 中的东西。


# References
- CRD in Helm, https://helm.sh/docs/topics/charts/#custom-resource-definitions-crds
