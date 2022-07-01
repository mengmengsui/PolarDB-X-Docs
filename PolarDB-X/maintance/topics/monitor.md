# 数据库监控

本小节介绍如何在Kubernetes集群中为PolarDB-X数据库开启监控功能。

## 安装PolarDB-X Monitor

PolarDB-X通过Prometheus和Grafana来监控PolarDB-X集群。PolarDB-X Monitor集成了[kube-promethus](https://github.com/prometheus-operator/kube-prometheus)组件栈。通过安装PolarDB-X Monitor即可一键部署监控所需的资源和组件。

### 前置条件

- 已经准备好一个运行中的Kubernetes集群，并确保集群版本为1.18.0或以上。

- 已经安装好[Helm 3](https://helm.sh/docs/intro/install/)。

- 已经安装好1.2.0及以上版本的PolarDB-X Operator。

### 安装Helm包

1. 首先创建一个名为polardbx-monitor的命名空间：

```bash
kubectl create namespace polardbx-monitor
```

2. 安装PolarDB-X Monitor CRD：

```bash
kubectl apply -f https://raw.githubusercontent.com/ApsaraDB/galaxykube/v1.2.1/charts/polardbx-operator/crds/polardbx.aliyun.com_polardbxmonitors.yaml
```

> **注意**：
>
> - 如果您的PolarDB-X Operator 1.2.0是通过helm install直接安装的，PolarDB-X Monitor的CRD会默认安装，可跳过该步骤。如果您的PolarDB-X Operator是从1.1.0及以下的低版本通过helm upgrade升级而来，需要执行如下命令手工安装PolarDB-X Monitor：
>
> ```bash
> helm install --namespace polardbx-monitor polardbx-monitor https://github.com/ApsaraDB/galaxykube/releases/download/v1.2.1/polardbx-monitor-1.2.1.tgz
> ```
>
> - 您也可以通过 PolarDB-X 的 Helm Chart 仓库安装：
>
> ```shell
> helm repo add polardbx https://polardbx-charts.oss-cn-beijing.aliyuncs.com
> helm install --namespace polardbx-monitor polardbx-monitor polardbx/polardbx-monitor
> ```
>
> - 通过这种方式安装Prometheus和Grafana采用的都是默认配置，便于快速体验。如果部署在生产集群，您可以参考[定制Prometheus和Grafana配置](monitor.md#定制Prometheus和Grafana配置)。
> - 如果您是在minikube上安装PolarDB-X Monitor, 可能会因为资源不足导致组件无法创建，可以参考[配置Prometheus和Grafana规格](monitor.md#配置Prometheus和Grafana规格)调整组件的规格。

预期将看到如下输出：

```bash
polardbx-operator monitor plugin is installed. Please check the status of components:

    kubectl get pods --namespace {{ .Release.Namespace }}

Now start to monitor your polardbx cluster.
```

3. PolarDB-X Monitor安装完成后，会在您Kubernetes集群的polardbx-monitor命名空间下创建Prometheus和Grafana等组件，以此来监控Kubernetes内的PolarDB-X。您可以通过如下命令检查相关组件是否正常，确认所有的Pod都处于Running状态。

```bash
kubectl get pods -n polardbx-monitor
```

## 开启PolarDB-X监控

PolarDB-X集群的监控采集功能默认是关闭的。需要为您需要监控的PolarDB-X Cluster创建PolarDB-X Monitor对象进行开启。

```bash
kubectl apply -f polardbx-monitor.yaml
```

其中polardbx-monitor.yaml的描述如下：

```yaml
apiVersion: polardbx.aliyun.com/v1
kind: PolarDBXMonitor
metadata:
  name: quick-start-monitor
spec:
  clusterName: quick-start
  monitorInterval: 30s
  scrapeTimeout: 10s
```

- spec.clusterName：需要开启监控的PolarDB-X集群名称
- spec.monitorInterval：监控数据采集频率，默认30s
- spec.scrapeTimeout：监控数据采集的超时时间，默认10s

## 访问Grafana Dashboard

默认情况下，执行如下命令将Grafana端口转发到本地：

```bash
kubectl port-forward svc/grafana -n polardbx-monitor 3000
```

在浏览器中输入<http://localhost:3000>，即可访问到PolarDB-X Dashboard，默认的用户名和密码都是admin。

> **注意**：由于Grafana的配置存储在ConfigMap中，您在Grafana中修改的密码或者新增的Dashboard不会被持久化，一旦Grafana Pod重建，这部分配置会丢失，请注意提前保存。

![Grafana](../images/monitor.png)

如果您的Kubernetes集群中支持LoadBalancer，您可以为Grafana的Service配置LoadBalancer进行访问，详见：[配置LoadBalancer](monitor.md#配置LoadBalancer)。

如果您的Kubernets集群内有多个PolarDB-X Cluster，可以通过Grafana页面上面的下拉框切换Namespace和PolarDB-X Cluster。

## 访问Prometheus

默认情况下，执行如下命令将Prometheus端口转发到本地：

```bash
kubectl port-forward svc/prometheus-k8s -n polardbx-monitor 9090
```

在浏览器中输入<http://localhost:9090>，即可访问到Prometheus页面。

如果您的Kubernetes集群中支持LoadBalancer，您可以为Prometheus的Service配置LoadBalancer进行访问，详见[配置LoadBalancer](monitor.md#配置LoadBalancer)。

## 定制Prometheus和Grafana配置

PolarDB-X Monitor的helm chart采用了默认的Prometheus和Grafana配置，如果您想修改相关配置，可以使用如下的命令安装或者升级PolarDB-X Monitor，通过values.yaml覆盖默认的配置。

```bash
helm install --namespace polardbx-monitor polardbx-monitor polardbx-monitor-1.2.0.tgz -f values.yaml
```

或者：

```bash
helm upgrade --namespace polardbx-monitor polardbx-monitor polardbx-monitor-1.2.0.tgz -f values.yaml
```

values.yaml文件包含了Prometheus和Grafana的相关配置项，下面针对常见的几种场景给出配置示例，详细的配置列表详见[values.yaml](https://raw.githubusercontent.com/ApsaraDB/galaxykube/v1.2.0/charts/polardbx-monitor/values.yaml)。

### 配置LoadBalancer

如果您的Kubernetes集群支持LoadBalancer，可以在安装或者升级PolarDB-X Monitor的时候通过`-f`参数指定如下配置：

```yaml
monitors:
  grafana:
    serviceType: LoadBalancer
  prometheus:
    serviceType: LoadBalancer
```

### 持久化监控数据

默认配置创建的Prometheus集群的监控数据是不持久化的，存在数据丢失的风险，您可以通过如下的values.yaml指定数据持久化的目录

```yaml
monitors:
  prometheus:
    persist: true
    # Kubernetes集群内支持的storage class
    storageClassName: ssd
    # 存储空间的大小
    storageRequest: 100G
```

### 配置Prometheus和Grafana规格

默认配置中，Prometheus集群包含1个节点，每个节点限定8C16G资源，Grafana包含1个节点，每个节点限定4C8G的资源。您可以通过如下配置项修改Prometheus和Grafana集群的规格和节点数量：

```yaml
monitors:
  grafana:
    resources:
      requests:
        cpu: 1000m
        memory: 2Gi
      limits:
        cpu: 2000m
        memory: 8Gi
  prometheus:
    resources:
      requests:
        cpu: 1000m
        memory: 2Gi
      limits:
        cpu: 2000m
        memory: 8Gi
```