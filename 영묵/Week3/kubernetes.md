# Taints

In Kubernetes, taints are used to mark nodes with specific attributes that repel certain pods. Taints allow administrators to specify that certain nodes should not accept pods unless those pods have a corresponding toleration.

A taint consists of a key, value, and effect. The key and value are arbitrary strings that can be used to identify the taint. The effect specifies how the taint affects pods that do not tolerate it. There are three possible effects:

NoSchedule: Pods will not be scheduled on the node unless they have a toleration for the taint.
PreferNoSchedule: Kubernetes will try to avoid scheduling pods on the node, but will not prevent it if there are no other options.
NoExecute: Existing pods on the node that do not tolerate the taint will be evicted immediately.
Taints can be added or removed using the kubectl taint command. For example, to add a taint with the key mytaint and the value NoSchedule to a node with the name mynode, you can use the following command:

```
kubectl taint nodes mynode mytaint=NoSchedule
```

To remove the same taint from the node, you can use the following command:

kubectl taint nodes mynode mytaint-

Taints can be useful in a variety of scenarios, such as ensuring that nodes with certain characteristics (such as GPU resources) are only used for specific workloads, or isolating nodes that are undergoing maintenance or updates.

### Get Taints

```
root@master:~/manifests# kubectl describe nodes | grep -A 3 Taints
Taints:             <none>
Unschedulable:      false
Lease:
  HolderIdentity:  master.youngmki.com
--
Taints:             <none>
Unschedulable:      false
Lease:
  HolderIdentity:  node1.youngmki.com
--
Taints:             <none>
Unschedulable:      false
Lease:
  HolderIdentity:  node2.youngmki.com
```