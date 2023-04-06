# What are taints and tolerations?

Taints are a Kubernetes node property that enable nodes to repel certain pods. Tolerations are a Kubernetes pod property that allow a pod to be scheduled on a node with a matching taint.

Let’s use an example to demonstrate the benefits of taints and tolerations. Consider a three-node cluster that consists of a master node and two worker nodes (node A and node B). Worker node A has an attached GPU, making it expensive but well suited for GPU-accelerated workloads while the worker node B is general purpose node without an attached GPU.

By adding a taint to the worker node A and only giving the necessary toleration to the GPU specific workloads, administrators can ensure that Kubernetes will only schedule workloads that can be accelerated with a GPU in the worker node A. All other pods will be scheduled on our general purpose worker node B. This technique ensures that valuable GPU capacity isn’t wasted in the underlying hardware by the wrong types of workloads.

![Tux, the Linux mascot](https://blog.kubecost.com/assets/images/kubernetes-taints/image_0.png)

## How to add Kubernetes taints

The kubectl taint command with the required taint allows us to add taints to nodes. The general syntax for the command is:

```


kubectl taint nodes <node name> <taint key>=<taint value>:<taint effect>




```

**NoSchedule** The pod will not get scheduled to the node without a matching toleration.

**NoExecute** This will immediately evict all the pods without the matching toleration from the node.

**PerferNoSchedule** This is a softer version of NoSchedule where the controller will not try to schedule a pod with the tainted node. However, it is not a strict requirement.

```



kubectl taint nodes minikube-m02 gpu=true:NoSchedule


```

for more information https://blog.kubecost.com/blog/kubernetes-taints/
