---
title: "使用 kubebuilder 开发一个 Zookeeper Operator"
date: 2022-02-13T18:59:25+08:00
lastmod: 2022-02-13T18:59:25+08:00
draft: true
keywords: []
tags: ["Kubernetes"]
categories: ["CloudNative"]
author: "Max"
toc: true
comment: true
autoCollapseToc: false
---

# 安装 kubebuilder

```
brew install kubebuilder
```

# 开发流程

1. 项目初始化

```
➜  kubebuilder init --domain atmax.io --owner "Max Xu" --repo github.com/jsonbruce/zookeeper-operator
Writing kustomize manifests for you to edit...
Writing scaffold for you to edit...
Get controller runtime:
$ go get sigs.k8s.io/controller-runtime@v0.11.0
Update dependencies:
$ go mod tidy
Next: define a resource with:
$ kubebuilder create api
```

2. 创建 GVK

```
➜  kubebuilder create api --group zookeeper --version v1alpha1 --kind ZookeeperCluster
Create Resource [y/n]
y
Create Controller [y/n]
y
Writing kustomize manifests for you to edit...
Writing scaffold for you to edit...
api/v1alpha1/zookeepercluster_types.go
controllers/zookeepercluster_controller.go
Update dependencies:
$ go mod tidy
Running make:
$ make generate
go: creating new go.mod: module tmp
Downloading sigs.k8s.io/controller-tools/cmd/controller-gen@v0.8.0
go get: installing executables with 'go get' in module mode is deprecated.
        To adjust and download dependencies of the current module, use 'go get -d'.
        To install using requirements of the current module, use 'go install'.
        To install ignoring the current module, use 'go install' with a version,
        like 'go install example.com/cmd@latest'.
        For more information, see https://golang.org/doc/go-get-install-deprecation
        or run 'go help get' or 'go help install'.
go get: added github.com/fatih/color v1.12.0
go get: added github.com/go-logr/logr v1.2.0
go get: added github.com/gobuffalo/flect v0.2.3
go get: added github.com/gogo/protobuf v1.3.2
go get: added github.com/google/go-cmp v0.5.6
go get: added github.com/google/gofuzz v1.1.0
go get: added github.com/inconshreveable/mousetrap v1.0.0
go get: added github.com/json-iterator/go v1.1.12
go get: added github.com/mattn/go-colorable v0.1.8
go get: added github.com/mattn/go-isatty v0.0.12
go get: added github.com/modern-go/concurrent v0.0.0-20180306012644-bacd9c7ef1dd
go get: added github.com/modern-go/reflect2 v1.0.2
go get: added github.com/spf13/cobra v1.2.1
go get: added github.com/spf13/pflag v1.0.5
go get: added golang.org/x/mod v0.4.2
go get: added golang.org/x/net v0.0.0-20210825183410-e898025ed96a
go get: added golang.org/x/sys v0.0.0-20210831042530-f4d43177bf5e
go get: added golang.org/x/text v0.3.7
go get: added golang.org/x/tools v0.1.6-0.20210820212750-d4cc65f0b2ff
go get: added golang.org/x/xerrors v0.0.0-20200804184101-5ec99f83aff1
go get: added gopkg.in/inf.v0 v0.9.1
go get: added gopkg.in/yaml.v2 v2.4.0
go get: added gopkg.in/yaml.v3 v3.0.0-20210107192922-496545a6307b
go get: added k8s.io/api v0.23.0
go get: added k8s.io/apiextensions-apiserver v0.23.0
go get: added k8s.io/apimachinery v0.23.0
go get: added k8s.io/klog/v2 v2.30.0
go get: added k8s.io/utils v0.0.0-20210930125809-cb0fa318a74b
go get: added sigs.k8s.io/controller-tools v0.8.0
go get: added sigs.k8s.io/json v0.0.0-20211020170558-c049b76a60c6
go get: added sigs.k8s.io/structured-merge-diff/v4 v4.1.2
go get: added sigs.k8s.io/yaml v1.3.0
/Users/max/Develop/Project/zookeeper-operator/bin/controller-gen object:headerFile="hack/boilerplate.go.txt" paths="./..."
Next: implement your new API and generate the manifests (e.g. CRDs,CRs) with:
$ make manifests
```

# 配置 RBAC
```
//+kubebuilder:rbac:groups="",resources=pods;services,verbs=get;list;watch;create;update;patch;delete
//+kubebuilder:rbac:groups=apps,resources=statefulsets,verbs=get;list;watch;create;update;patch;delete
//+kubebuilder:rbac:groups=zookeeper.atmax.io,resources=zookeeperclusters,verbs=get;list;watch;create;update;patch;delete
//+kubebuilder:rbac:groups=zookeeper.atmax.io,resources=zookeeperclusters/status,verbs=get;update;patch
//+kubebuilder:rbac:groups=zookeeper.atmax.io,resources=zookeeperclusters/finalizers,verbs=update
```

否则生产环境的 operator 会因为权限不足无法读写相关资源。

# FAQ
1. Reconcile 函数无限循环地 Requeue, 是否会对用户触发新的 Create/Update/Delete 事件有影响？

2. Operator 的工作原理？既然是拿的缓存，怎么实时感知到新事件的？

3. 从 kubebuilder 分析 client-go 原理



# k8s.io/client-go
> github.com/kubernetes/kubernetes/staging/src/k8s.io/client-go

Reflector -> DeltaFIFO -> Indexer -> Workqueue -> Reconciler



# sigs.k8s.io/controller-runtime

# 使用 StatefulSet 部署 Zookeeper
> https://kubernetes.io/docs/tutorials/stateful-application/zookeeper/

```
apiVersion: v1
kind: Service
metadata:
  name: zk-hs
  labels:
    app: zk
spec:
  ports:
  - port: 2888
    name: server
  - port: 3888
    name: leader-election
  clusterIP: None
  selector:
    app: zk
---
apiVersion: v1
kind: Service
metadata:
  name: zk-cs
  labels:
    app: zk
spec:
  ports:
  - port: 2181
    name: client
  selector:
    app: zk
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: zk-pdb
spec:
  selector:
    matchLabels:
      app: zk
  maxUnavailable: 1
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: zk
spec:
  selector:
    matchLabels:
      app: zk
  serviceName: zk-hs
  replicas: 3
  updateStrategy:
    type: RollingUpdate
  podManagementPolicy: OrderedReady
  template:
    metadata:
      labels:
        app: zk
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: "app"
                    operator: In
                    values:
                    - zk
              topologyKey: "kubernetes.io/hostname"
      containers:
      - name: kubernetes-zookeeper
        imagePullPolicy: Always
        image: "k8s.gcr.io/kubernetes-zookeeper:1.0-3.4.10"
        resources:
          requests:
            memory: "1Gi"
            cpu: "0.5"
        ports:
        - containerPort: 2181
          name: client
        - containerPort: 2888
          name: server
        - containerPort: 3888
          name: leader-election
        command:
        - sh
        - -c
        - "start-zookeeper \
          --servers=3 \
          --data_dir=/var/lib/zookeeper/data \
          --data_log_dir=/var/lib/zookeeper/data/log \
          --conf_dir=/opt/zookeeper/conf \
          --client_port=2181 \
          --election_port=3888 \
          --server_port=2888 \
          --tick_time=2000 \
          --init_limit=10 \
          --sync_limit=5 \
          --heap=512M \
          --max_client_cnxns=60 \
          --snap_retain_count=3 \
          --purge_interval=12 \
          --max_session_timeout=40000 \
          --min_session_timeout=4000 \
          --log_level=INFO"
        readinessProbe:
          exec:
            command:
            - sh
            - -c
            - "zookeeper-ready 2181"
          initialDelaySeconds: 10
          timeoutSeconds: 5
        livenessProbe:
          exec:
            command:
            - sh
            - -c
            - "zookeeper-ready 2181"
          initialDelaySeconds: 10
          timeoutSeconds: 5
        volumeMounts:
        - name: datadir
          mountPath: /var/lib/zookeeper
      securityContext:
        runAsUser: 1000
        fsGroup: 1000
  volumeClaimTemplates:
  - metadata:
      name: datadir
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```

# Zookeeper 介绍
## 端口

- `2181`: 客户端连接 ZooKeeper 集群使用的监听端口号
- `2888`: 集群内通信。Leader 和 Follower 之间数据同步使用的端口号
- `3888`: 集群内通信。Leader 选举专用的端口号
- `8080`: AdminServer 端口 (zk 3.5 版本新特性)

# Troubleshooting
## Zookeeper hostname resolution fails
- Zookeeper: Hostname resolution fails, https://stackoverflow.com/questions/46605686/zookeeper-hostname-resolution-fails
- Zookeeper ensemble name resolution issues at startup, https://github.com/confluentinc/cp-helm-charts/issues/205
- Cluster doesn't fully start, https://github.com/confluentinc/cp-helm-charts/issues/503

使用 `Lifecycle` 的 `PostStart` hook 解决此问题

```
PostStart: &corev1.LifecycleHandler{
				Exec: &corev1.ExecAction{
					Command: []string{"/bin/sh", "-c", "echo ${HOSTNAME##*-} > ${ZOO_DATA_DIR}/myid && sed -i -e \"s/$(hostname -f)/0.0.0.0/g\" ${ZOO_CONF_DIR}/zoo.cfg"},
				},
			}
```

# 参考
- 使用 Operator 对 Kubernetes 扩展, https://lailin.xyz/post/operator-01-overview.html
- Kubernetes Operators Best Practices, https://cloud.redhat.com/blog/kubernetes-operators-best-practices
- Deploying Zookeeper Cluster, https://medium.com/kafka-in-kubernetes/deploying-zookeeper-cluster-3acdcc7ed340
- How to force NOT to call Reconcile func again after updating the status subresource? https://github.com/kubernetes-sigs/kubebuilder/issues/618
- Fire reconcile() on a timer? https://github.com/kubernetes-sigs/kubebuilder/issues/1131
