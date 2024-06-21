## Kubernetes Scheduling

- In Kubernetes, scheduling refers to making sure that Pods are matched to Nodes so that Kubelet can run them.

- A scheduler watches for newly created Pods that have no Node assigned. 
- For every Pod that the scheduler discovers, the scheduler becomes responsible for finding the best Node for that Pod to run on. 

#### Kube Schedular

- kube-scheduler is the default scheduler for Kubernetes and runs as part of the control plane.

### Node selection

kube-scheduler selects a node for the pod in a 2-step operation:

- Filtering: 

The filtering step finds the set of Nodes where it's feasible to schedule the Pod.  eg, the `PodFitsResources` filter checks whether a candidate Node has enough available resources to meet a Pod's specific resource requests.

- Scoring:

In the scoring step, the scheduler ranks the remaining nodes to choose the most suitable Pod placement. The scheduler assigns a score to each Node that survived filtering, basing this score on the active scoring rules.

Finally, kube-scheduler assigns the Pod to the Node with the highest ranking.

### Mechanism to Assign Pods to Nodes

You can use any of the following methods to choose where Kubernetes schedules specific Pods:

- `nodeSelector` field matching against `node labels`
- `Affinity` and `anti-affinity`
- `nodeName` field
- Pod `topology` spread `constraints`

### Node Selector

`nodeSelector` is the simplest recommended form of node selection constraint. You can add the `nodeSelector` field to your Pod specification and specify the node labels you want the target node to have. [example](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/)


### Affinity and anti-affinity

Some of the benefits of affinity and anti-affinity include:

- `nodeSelector` only selects nodes with all the specified labels. Affinity/anti-affinity gives you more control over the selection logic.
- You can indicate that a rule is `soft or preferred`, so that the scheduler still schedules the Pod even if it can't find a matching node.
- You can constrain a Pod using labels on other Pods running on the node (or other topological domain), instead of just node labels, which allows you to define rules for which Pods can be co-located on a node.

The affinity feature consists of two types of affinity:
- `Node affinity` functions like the nodeSelector field but is more expressive and allows you to specify soft rules.
- `Inter-pod affinity/anti-affinity` allows you to constrain Pods against labels on other Pods.


#### Node Affinity

Defined under `.spec.affinity.nodeAffinity` property.

There are two types of node affinity:
- `requiredDuringSchedulingIgnoredDuringExecution`: Like nodeSelector, but with a more expressive syntax.
- `preferredDuringSchedulingIgnoredDuringExecution`

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: topology.kubernetes.io/zone
            operator: In #https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#operators
            values:
            - antarctica-east1
            - antarctica-west1
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In #https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#operators
            values:
            - another-node-label-value
  containers:
  - name: with-node-affinity
    image: registry.k8s.io/pause:2.0
```

NOTE:
- If you specify both `nodeSelector` and `nodeAffinity`, both must be satisfied for the Pod to be scheduled onto a node.
- If you specify multiple terms in `nodeSelectorTerms` associated with nodeAffinity types, then the Pod can be scheduled onto a node if one of the specified terms can be satisfied (terms are ORed).
- If you specify multiple expressions in a single `matchExpressions` field associated with a term in `nodeSelectorTerms`, then the Pod can be scheduled onto a node only if all the expressions are satisfied (expressions are ANDed).

##### Node affinity per scheduling profile

The addedAffinity is applied to all Pods that set `.spec.schedulerName` to `foo-scheduler`, in addition to the `NodeAffinity` specified in the PodSpec.

```yaml
apiVersion: kubescheduler.config.k8s.io/v1beta3
kind: KubeSchedulerConfiguration

profiles:
  - schedulerName: default-scheduler
  - schedulerName: foo-scheduler
    pluginConfig:
      - name: NodeAffinity
        args:
          addedAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
              nodeSelectorTerms:
              - matchExpressions:
                - key: scheduler-profile
                  operator: In
                  values:
                  - foo
```

NOTE:

- The DaemonSet does not support scheduling profiles. When the DaemonSet controller creates Pods, the default Kubernetes scheduler places those Pods and honors any `nodeAffinity` rules in the DaemonSet controller.

#### Inter-pod affinity and anti-affinity 

Inter-pod `affinity` and `anti-affinity` allow you to constrain which nodes your Pods can be scheduled on based on the labels of Pods already running on that node, instead of the node labels.


There are two types of node affinity:
- `requiredDuringSchedulingIgnoredDuringExecution`
- `preferredDuringSchedulingIgnoredDuringExecution`

Example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-affinity
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: security
            operator: In
            values:
            - S1
        topologyKey: topology.kubernetes.io/zone
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: security
              operator: In
              values:
              - S2
          topologyKey: topology.kubernetes.io/zone
  containers:
  - name: with-pod-affinity
    image: registry.k8s.io/pause:2.0
```
NOTES:

- For Pod `affinity` and `anti-affinity`, an empty `topologyKey field` is not allowed in both `requiredDuringSchedulingIgnoredDuringExecution` and `preferredDuringSchedulingIgnoredDuringExecution`.


### nodeName

nodeName is a more direct form of node selection than `affinity` or `nodeSelector`. nodeName can configured using `.spec.nodeName` of the Pod spec. 

If the nodeName field is not empty, the scheduler ignores the Pod and the kubelet on the named node tries to place the Pod on that node. Using nodeName overrules using `nodeSelector` or `affinity` and `anti-affinity` rules.


### Pod `topology` spread `constraints`

https://kubernetes.io/docs/concepts/scheduling-eviction/topology-spread-constraints/

## Taints and Tolerations

https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/