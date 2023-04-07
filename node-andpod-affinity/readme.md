# Node Affinity

Node affinity can be used for advanced scheduling in kubernetes. Tutorial below is a walk through of such usage.

## Concepts

### Node Affinity

- Node affinity is a way to set rules based on which the scheduler can select the nodes for scheduling workload. Node affinity can be thought of as opposite of taints. Taints repel a certain set of nodes where as node affinity attract a certain set of nodes.

- NodeAffinity is a generalization of _nodeSelector_. In _nodeSelector_, we specifically mention which node the pod should go to, using node affinity we specify certain rules to select nodes on which pod can be scheduled.

- These rules are defined by labeling the nodes and having pod spec specify the selectors to match those labels. There are 2 types of affinity rules. Preferred rules and Required rules.

- In _Preferred rule_, a pod will be assigned on a non matching node if and only if no other node in the cluster matches the specified labels. _preferredDuringSchedulingIgnoredDuringExecution_ is a preferred rule affinity.

- In _Required rules_, if there are no matching nodes, then the pod won't be scheduled. There are a couple of require rule affinities namely _requiredDuringSchedulingIgnoredDuringExecution_ and _requiredDuringSchedulingRequiredDuringExecution_.

- In _requiredDuringSchedulingIgnoredDuringExecution_ affinity, a pod will be scheduled only if the node labels specified in the pod spec matches with the labels on the node. However, once the pod is scheduled, labels are ignored meaning even if the node labels change, the pod will continue to run on that node.

- In _requiredDuringSchedulingRequiredDuringExecution_ affinity, a pod will be scheduled only if the node labels specified in the pod spec matches with the labels on the node and if the labels on the node change in future, the pod will be evicted. This effect is similar to _NoExecute_ taint with one significant difference. When _NoExecute_ taint is applied on a node, every pod not having a toleration will be evicted, where as, removing/changing a label will remove only the pods that do specify a different label.

## Use cases

- While scheduling workload, when we need to schedule a certain set of pods on a certain set of nodes but do not want those nodes to reject everything else, using node affinity makes sense.

## Examples:

Follow through guide.

Let's begin with listing nodes.

```
kubectl get nodes
```

You should be able to see the list of nodes available in the cluster,

```
NAME                          STATUS    ROLES     AGE       VERSION
node1.compute.infracloud.io   Ready     <none>    25m       v1.9.4
node2.compute.infracloud.io   Ready     <none>    25m       v1.9.4
node3.compute.infracloud.io   Ready     <none>    28m       v1.9.4
```

NodeAffinity works on label matching. Let's label node1 as,

```
kubectl label nodes node1.compute.infracloud.io node1=TheChosenOne
```

Make sure that you are able to see this label applied to the node,

```
kubectl get nodes --show-labels | grep TheChosenOne
```

Now let's try to deploy the entire guestbook on the node1. In all the [deployment yaml files](guestbook/), a NodeAffinity for node1 is added as,

```
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: "node1"
                operator: In
                values: ["TheChosenOne"]
```

In a couple of minutes, you should be able to see that all the pods are scheduled on node1.

```
NAME                            READY     STATUS    RESTARTS   AGE       IP            NODE
frontend-85b968cdc5-c785v       1/1       Running   0          49s       10.20.29.13   node1.compute.infracloud.io
frontend-85b968cdc5-pw2kl       1/1       Running   0          49s       10.20.29.14   node1.compute.infracloud.io
frontend-85b968cdc5-xxh7h       1/1       Running   0          49s       10.20.29.15   node1.compute.infracloud.io
redis-master-7bbf6b76bf-ttb6b   1/1       Running   0          1m        10.20.29.10   node1.compute.infracloud.io
redis-slave-747f8bc7c5-2tjtw    1/1       Running   0          1m        10.20.29.11   node1.compute.infracloud.io
redis-slave-747f8bc7c5-clxzh    1/1       Running   0          1m        10.20.29.12   node1.compute.infracloud.io
```

The output will also yield a load balancer ingress url which can be used to load the guestbook.

# Pod Affinity & AntiAffinity

Pod affinity can be used for advanced scheduling in kubernetes. Tutorial below is a walk through of such usage.

## Concepts

### Pod Affinity & AntiAffinity

- Node affinity allows you to schedule a pod on a set of nodes based on labels present on the nodes. However, in certain scenarios, we might want to schedule certain pods together or we might want to make sure that certain pods are never scheduled together. This can be achieved by _PodAffinity_ and/or _PodAntiAffinity_ respectively.

- Similar to node affinity, there are a couple of variants in pod affinity namely _requiredDuringSchedulingIgnoredDuringExecution_ and _preferredDuringSchedulingIgnoredDuringExecution_.

## Use cases

- While scheduling workload, when we need to schedule a certain set of pods together, _PodAffinity_ makes sense. Example, a web server and a cache.
- While scheduling workload, when we need to make sure that a certain set of pods are not scheduled together, _PodAntiAffinity_ makes sense.

## Examples:

Follow through guide.

Let's begin with listing nodes.

```
kubectl get nodes
```

You should be able to see the list of nodes available in the cluster,

```
NAME                          STATUS    ROLES     AGE       VERSION
node1.compute.infracloud.io   Ready     <none>    25m       v1.9.4
node2.compute.infracloud.io   Ready     <none>    25m       v1.9.4
node3.compute.infracloud.io   Ready     <none>    28m       v1.9.4
```

#### Pod Affinity example

Let's deploy [deployment-Affinity.yaml](podaffinity.yaml), which has pod affinity as,

```
affinity:
  podAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values:
          - nginx
      topologyKey: "kubernetes.io/hostname"
```

Here we are specifying that all nginx pods should be scheduled together.

```
kubectl apply -f deployment-Affinity.yaml
```

Check the pods using,

```
kubectl get pods -o wide -w
```

You should be able to see that all pods are scheduled on the same node.

```
NAME                                READY     STATUS    RESTARTS   AGE       IP            NODE
nginx-deployment-6bc5bb7f45-49dtg   1/1       Running   0          36m       10.20.29.18   node2.compute.infracloud.io
nginx-deployment-6bc5bb7f45-4ngvr   1/1       Running   0          36m       10.20.29.20   node2.compute.infracloud.io
nginx-deployment-6bc5bb7f45-lppkn   1/1       Running   0          36m       10.20.29.19   node2.compute.infracloud.io
```

To clean up run,

```
kubectl delete -f deployment-Affinity.yaml
```

#### Pod Anti Affinity example

Let's deploy [deployment-AntiAffinity.yaml](antiaffinity.yaml), which has pod affinity as,

```
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values:
          - nginx
      topologyKey: "kubernetes.io/hostname"
```

Here we are specifying that no two nginx pods should be scheduled together.

```
kubectl apply -f deployment-AntiAffinity.yaml
```

Check the pods using,

```
kubectl get pods -o wide -w
```

You should be able to see that pods are scheduled on different nodes.

```
NAME                                READY     STATUS    RESTARTS   AGE       IP            NODE
nginx-deployment-85d87bccff-4w7tf   1/1       Running   0          27s       10.20.29.16   node3.compute.infracloud.io
nginx-deployment-85d87bccff-7fn47   1/1       Running   0          27s       10.20.42.32   node1.compute.infracloud.io
nginx-deployment-85d87bccff-sd4lp   1/1       Running   0          27s       10.20.13.17   node2.compute.infracloud.io

```

Note: In above example, if number of replicas are more than number of nodes then some pods will remain in pending state.

To clean up run,
