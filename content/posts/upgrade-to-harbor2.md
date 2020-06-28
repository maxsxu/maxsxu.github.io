+++ 
draft = false
date = 2020-06-28T16:19:24+08:00
title = "升级到 Harbor 2.0.0"
description = ""
slug = "" 
tags = ["Harbor"]
categories = ["CloudNative"]
externalLink = ""
series = []
+++


# 前言
早就看到 Harbor v2 GA 了，但线上业务在正常跑也就没有管，上周 Harbor 正式从 CNCF 毕业了（第 11 个毕业毕业项目），我想是时候拥抱变化了。

# 升级
线上的 Harbor v1 是用 Helm 部署在 K8s 集群。升级很简单:

```
# 首先更新 Chart Repo
helm repo update

# 执行升级
helm upgrade --install harbor harbor/harbor
```

Harbor 的 database 和 redis 组件是 StatefulSet，存储接的 PVC，所以升级对数据没有影响，不用担心。

# 新特性
## OCI 规范的支持
这应该是最激动的一个特性，统一。

现在 Docker Image, Helm Chart, CNAB (Cloud Native Application Bundle) 都可以放在同一个 Project->Repository 里了。
Harbor 1.0 的 Helm Chart 是放在 Project->Helm Chart 里，升级后原有的 Chart 依然放在这里。
总结来看 Harbor 2.0 的 Helm Chart 有两种存储方式：

- 存储在 Project -> Helm Chart. 使用 helm push 插件和 helm pull 命令
- 存储在 Project -> Repository. 使用 Helm v3 的实验特性 helm chart pull 和 helm chart push 命令

Helm Chart OCI Registry 的存储目录为 `~/.cache/helm/registry`

```
~/.cache/helm/registry/
├── cache/
│   ├── blobs/
│   │   └── sha256/
│   │       ├── 0ebcb18716bb0f7778914315255d561deecb7d1d6c8691f949070b4252eacfc9
│   │       ├── 5202f80d4df471a986cbea192f331c6f4d5f5e36a617f1102042faade2e0ba54
│   │       ├── 847317a2b1a500082da4f2d62da66acd95950b14f45704ec133042623db9f662
│   │       ├── 85c5ee407074291d4b6686d85347cc295ecc973d17e1fee9358493beb004dacb
│   │       ├── 8dd036c9a8f4b5c9934047f5e8a3a5f4edac9252e0588d6a9fecf060a9e3f141
│   │       ├── e00492b73b6e18b24d880df5e4d3043e59e807e10ccd2945eca9f09773293fbd
│   │       └── fd07ebfefb5b619878fd9a4af0bac249241127cfbb59e574a2e9b3366dc0ed8c
│   ├── .build/
│   ├── ingest/
│   ├── index.json
│   └── oci-layout
└── config.json
```

其中 `index.json` 文件包含了对所有 Helm Chart 清单的引用：

```
$ cat ~/.cache/helm/registry/cache/index.json | jq
{
  "schemaVersion": 2,
  "manifests": [
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "digest": "sha256:fd07ebfefb5b619878fd9a4af0bac249241127cfbb59e574a2e9b3366dc0ed8c",
      "size": 322,
      "annotations": {
        "org.opencontainers.image.ref.name": "mirrors.jsonbruce.com/library/nginx-chart:0.1.0"
      }
    }
  ]
}
```

# 使用
## 操作 Helm Chart
要使用 OCI 规范的 Helm Chart, 要确保:

- Helm v3.0.0+ (本文使用 Helm v3.2.4)
- 由于目前在 Helm 中还是实验特性，所以需要开启 `HELM_EXPERIMENTAL_OCI=1` 环境变量

首先要登录 Harbor: `helm registry login mirrors.jsonbruce.com`

**上传**

```
helm chart save nginx-chart mirrors.jsonbruce.com/library/nginx-chart:0.1.0

helm chart push mirrors.jsonbruce.com/library/nginx-chart:0.1.0 
```

**拉取**

```
helm chart pull mirrors.jsonbruce.com/library/nginx-chart:0.1.0 
```

**安装**

目前还不能直接从 OCI Registry 安装

```
helm chart export mirrors.jsonbruce.com/library/nginx-chart:0.1.0
helm install nginx ./nginx-chart
```

# 问题
1. 统一存储的问题

当使用 OCI 存储时，Docker Image 和 Helm Chart 冲突。
假设首先 push 一个 Docker Image `docker push mirrors.jsonbruce/library/nginx:0.1.0`
然后 push 一个同名同标签的 Helm Chart `helm chart push mirrors.jsonbruce/library/nginx:0.1.0`

可以发现 Docker Image 的 tag 被覆盖了，变成一个没有 tag 的 Image。所以这儿应该需要一个机制避免这种情况。

2. Helm v3 的问题

目前无法直接从 OCI Registry 安装 Chart, 已经有人提了这个 Issue 了 [Install chart from registry](https://github.com/helm/helm/issues/6982) 社区相关讨论和 PR 也在进行中了，期待早日 GA.

同时也不支持通过 Helm 删除 OCI Registry 的 Chart. (helm chart remove 只支持删除本地 Cache 的 Chart)


# 参考
- Cloud Native Computing Foundation Announces Harbor Graduation, https://www.cncf.io/announcement/2020/06/23/cloud-native-computing-foundation-announces-harbor-graduation/
- Managing Helm Charts, https://goharbor.io/docs/2.0.0/working-with-projects/working-with-images/managing-helm-charts
- Helm Registries, https://helm.sh/docs/topics/registries/