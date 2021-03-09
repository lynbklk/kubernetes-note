## 1. Kubernetes Scheduler

### 1.1 kube-scheduler 分两步选取Node

- Filtering
- Scoring

### 1.2 kube-scheduler 性能调优

percentageOfNodesToScore: 可调度节点小于50时，此参数无效(100%)

### 1.3 Pod 拓扑分布约束

```yaml 
# pod spec
spec:
  topologySpreadConstraints:
    - maxSkew: <integer>
      topologyKey: <string>
      whenUnsatisfiable: <string>
      labelSelector: <object>
	  # topologKey: 节点label值，按照该值进行拓扑域划分
	  # whenUnsatisfiable： 
	  #		DoNotSchedule (default) 不满足约束时不进行调度
	  #		ScheduleAnyway 不满足约束时尽量使skew值最小
	  # maxSkew: 描述可接受Pod分布不均的程度
	  # labelSelector: 用于查找匹配Pod
```
约定：
- 只有与新的 Pod 具有相同命名空间的 Pod 才能作为匹配候选者
- 没有topologyKey 的节点将被忽略
- 如果新 Pod 定义了 spec.nodeSelector 或 spec.affinity.nodeAffinity，则 不匹配的节点会被忽略。

可以设置集群级别的默认约束  

