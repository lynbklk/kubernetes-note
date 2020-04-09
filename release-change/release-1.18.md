# k8s 1.17/1.18版本简介
主要是保证功能稳定性和新特性增强上

## 主要更新内容为：

### 1、Topology aware routing of services
* 状态：k8s 1.17引入，目前alpha
* 功能：即让 Service 可以实现就近转发，比如同节点、同 AZ、同 Region 

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
  topologyKeys:
  - "topology.kubernetes.io/zone"
  - "topology.kubernetes.io/region"
```
* 影响范围：在后续云平台支持svc之后，建议采用该策略，实现就近转发

### 2、Move Frequent Kubelet Heartbeats To Lease Api
* 状态：k8s 1.17为GA版本
* 功能：后续kubelet和apiserver同步状态，从nodestatus迁移到Lease同步
* 影响范围：可能对node-mutator有影响，需要评估一下


### 3、Configurable Pod Process Namespace Sharing
* 状态：k8s 1.17为GA版本
* 功能：同一个pod内的容器可以共享进程命名空间

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  shareProcessNamespace: true
  containers:
  - name: nginx
    image: nginx
  - name: shell
    image: busybox
    securityContext:
      capabilities:
        add:
        - SYS_PTRACE
    stdin: true
    tty: true

```
* 影响范围：在后续多容器需要进程间通信的场景下，可以使用该特性，web界面可以考虑提供该配置

### 4、Defaulting of Custom Resources
* 状态：k8s 1.17为GA版本
* 功能：可以在CRD定义为字段增添default值
* 影响范围：CRD定义

### 5、KubernetesCSI Topology Support
* 状态：k8s 1.17为GA版本
* 功能：支持存储拓扑调度感知

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
volumeBindingMode: WaitForFirstConsumer
allowedTopologies:
- matchLabelExpressions:
  - key: failure-domain.beta.kubernetes.io/zone
    values:
    - us-central1-a
    - us-central1-b
```
* 影响范围：在有状态服务使用存储，推荐采用这种模式

### 6、KubernetesTopology Manager
* 状态：k8s 1.18为Beta版本
* 功能：支持kubelet对硬件拓扑结构如CPU、GPU的感知，从而优化资源分配

```yaml
spec:
  containers:
  - name: numa-aligned-container
    image: alpine
    resources:
      limits:
        cpu: 2
        memory: 200Mi
        gpu-vendor.com/gpu: 1
        nic-vendor.com/nic: 1
```
* 影响范围：云平台后续要支持一些高性能计算应用/GPU应用，可以建议使用该特性

### 7、kubectl debug
* 状态：k8s 1.18为alpha版本
* 功能：kubectl debug命令可以让开发集群中调试其Pod。此命令允许创建一个临时容器，该容器在要检查的Pod旁运行，并且还附加到控制台以进行交互式故障排除。

```sh
kubectl alpha debug -it ephemeral-demo --image=busybox --target=ephemeral-demo
```
* 影响范围：目前临时容器还处在alpha阶段，不建议在生产环境使用


### 8、Pod拓扑约束(高可用调度)
* 状态：k8s 1.18为beta版本
* 功能：使用topologySpreadConstraints，可以定义规则，在多区域集群中均匀分布pods，保证高可用性，并提高资源利用率。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  topologySpreadConstraints:
  - maxSkew: <integer>
    topologyKey: <string>
    whenUnsatisfiable: <string>
    labelSelector: <object>
```
* 影响范围：可以显示的提供高可用调度配置规则，满足用户需求

### 9、Run multiple Scheduling Profiles
* 状态：k8s 1.18为alpha版本
* 功能：可以在kube-scheduler上配置多个调度器
* 影响范围：扩展自定义调度


### 10、HPA可配置伸缩速率
* 状态：k8s 1.18为alpha版本
* 功能：可以为HPA指定扩缩容的速率

```yaml
behavior:
  scaleUp:
    policies:
    - type: Percent
      value: 100
      periodSeconds: 15
  scaleDown:
    policies:
    - type: Pods
      value: 4
      periodSeconds: 60
```
* 影响范围：有比较多的应用场景，但目前该特性还在试验阶段，且只能配置在集群生效，建议持续跟进该特性演进

### 11、扩展Hugepage特性
* 状态：k8s 1.18为beta版本
* 功能：pod现在可以请求不同大小的HugePages。HugePages是一种机制，它可以预留具有预定义大小的大内存块。这对于处理内存中的大数据集或对内存访问延迟敏感的应用（如数据库或虚拟机）尤其有用。

```yaml
apiVersion: v1
kind: Pod
…
spec:
  containers:
…
    resources:
      requests:
        hugepages-2Mi: 1Gi
        hugepages-1Gi: 2Gi
```
* 影响范围：目前云平台节点未开启hugepage，后续有应用(如数据库)等需要hugepage特性时，可以再开启该特性。

### 12、 startupProbe
* 状态: k8s1.18为Beta版本
* 功能: 为了避免应用程序在启动时需要较多的初始化时间，造成探测死锁的快速响应，引入了一个启动探测器，只有启动探测器成功后，健康探针才会接管探测

```yaml
startupProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 30
  periodSeconds: 10
```
* 影响范围：在一些启动耗时较长的有状态服务，如需要加载大量数据的服务，建议使用该特性。

### 13、NodeLocal DNSCache
* 状态: k8s1.18为GA版本
* 功能：NodeLocalDNSCache通过在集群节点上以Daemonset运行DNS缓存代理来提高集群DNS性能，
* 影响范围：在后续容器云平台支持svc后，可以开启dns缓存，加快dns查询性能


### 14、服务帐户令牌发布方提供OIDC发现
* 状态：k8s1.18为alpha版本
* 功能： Kubernetes服务帐户(KSA)可以使用令牌(JSON Web令牌或JWT)对KubernetesAPI进行身份验证。
例如使用kubectl --token \<the_token_string\>.但是，Kubernetes API是目前唯一能够验证这些令牌的服务。
由于Kubernetes API服务器不能(也不应该)从公共网络访问，一些工作负载必须使用独立的系统进行身份验证。比如，跨集群身份验证。
此增强的目的是提高KSA令牌的实用性，允许集群之外的服务用作通用身份验证方法，而无需重载API服务器。
为实现这点，API服务器提供了一个OpenID Connect (OIDC)发现文档，其中包含令牌公钥和其他数据。验证者可以使用这些密钥来验证KSA令牌。
使用ServiceAccountIssuerDiscovery功能开关启用OIDC发现，并且进行一些配置使其工作。
* 影响范围：暂无

### 15、APIServer DryRun
* 状态：k8s1.18为GA版本


### 16. Sidecar容器
* 状态：sidecar容器推迟到k8s1.19版本发布
* 功能：使sidecar容器有了它特有的生命周期，解决pod启动、pod终结、及job执行结束可能遇到的问题。

**pod启动:**

改版之前，sidecar容器和主业务容器是在InitContainer之后一起无差别并行启动的，这就可能引入一些问题。
比如一个envoy sidecar负责了主业务容器的网络传输，在envoy sidecar ready之前，主业务容器是无网络，
这可能导致探针误判，错误日志输出等异常情况，在处理不合理的时候服务可能一直就是处于失败重启阶段。

使用Sidecar声明后，Sidecar容器会在InitContainer之后，主业务容器之前初始化并启动。
主业务容器会在sidecar容器ready后才会被拉起，规避以上的问题。

**pod终结：**

改版之前，sidecar容器和主业务容器同时并行销毁，如果sidecar容器先于主业务容器销毁，可能会
影响到主业务容器的一些回收处理逻辑的执行。

使用Sidecar声明后，pod会先发SIGTERM信号给主业务容器进行容器销毁工作，当主业务容器正常终结，或者
超过TerminationGracePeriod时间，才会发SIGTERM给Sidecar容器，规避以上问题。

**Job执行：**

改版之前，sidecar容器和主业务容器无区分，可能导致主业务容器Job跑完了，Sidecar容器还在跑，导致
Job一直也到不了Completed阶段。用户如果想让Job正常运行可能就需要自行实现容器之间的同步操作，让
主业务容器执行完成通知Sidecar容器终结。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: bookings-v1-b54bc7c9c-v42f6
  labels:
    app: demoapp
spec:
  containers:
  - name: bookings
    image: banzaicloud/allspark:0.1.1
    ...
  - name: istio-proxy
    image: docker.io/istio/proxyv2:1.4.3
    lifecycle:
      type: Sidecar #另外还有Standard类型
```
* 影响范围：在使用enovy，日志sidecar等容器时，可以显示声明为sidecar生命周期，保证在应用容器启动前启动完成

### 17、 Serverside Apply 
* 状态：beta2
* 功能：Serverside Apply是一个跑在apiServer端的合并算法，它还会记录field的所有权。
Serverside Apply会跟踪系统内用户对某一资源对象字段的修改，它会diff并且记录字段的更改及时间，
并保存在该对象的metadata的managedFields字段中。

* 使用方式：
kubectl apply --server-side （--force-conflicts加此字段会强制修改并且移交字段所有权）

* 影响范围：对协同多用户或多源操作同一资源可能有帮助。

### 18、将 qps 和 burst 作为 kubeconfig 的标准 api
* 状态：GA
* 功能：将 qps/burst 作为一种标准的 API,方便后续对客户端请求做控制
* 影响范围：认证分发kubeconfig，建议增加该限制。

## 19、 API版本升级
- apps/v1beta1和apps/v1beta2版本下的资源全部废弃，请使用apps/v1版本
- extensions/v1beta1下的daemonsets, deployments, replicasets等资源，请使用apps/v1版本
- extensions/v1beta1下的networkpolicies资源，请使用networking.k8s.io/v1版本
- extensions/v1beta1下的podsecuritypolicies资源，使用policy/v1beta1版本
以上老版本资源接口在1.18都不再支持。
