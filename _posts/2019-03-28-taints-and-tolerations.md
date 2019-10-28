---
layout: post
title:  "Use Taints and Tolerations in Kubernetes to repel pods from a worker node"
---

Recently, the Kubernetes cluster that I manage at work seems to be performing poorly. My colleagues are complaining that doing a search 
in Kibana takes minutes instead of seconds; the service running in the cluster takes much longer to complete a customer transction. 

We have noticed that ELK, Weavescope and Prometheus are very resource hungry, so we started to think is it possible to move all the monitoring apps to one particular node. 

I started by putting a `label` on a worker node. You can use the following command to attach a label on a node:

`kubectl label nodes <node-name> <label-key>=<label-value>`


then I attach a `nodeSelector` on the pod config of the monitoring apps so that they will get scheduled to that particular worker node. 

```
spec:
  containers:
  - name: grafana
    image: grafana
    imagePullPolicy: IfNotPresent
  nodeSelector:
    <label-key>: <label-value>
```

This works well, I can see that after I attached the nodeSelector, all the monitoring apps started moving to the node with the matching label. 

However, I can still see other pods (the non monitoring ones) getting scheduled to this node. So I started searching for ways to repel pods 
away from this node. I found that you can use `taints` in Kubernetes to achieve this. You can set a `taint` on a node just like how you can set a `label` on a node. This is the command to set a taint on a node

`kubectl taint nodes <node-name> <taint-key>=<taint-value>:NoSchedule`

where the `NoSchedule` keyword is an effect that basically means pods without the matching `toleration` cannot be scheduled on this node.


Toleration is something you can set on a pod just like `nodeSelectors`. To make pods scheduelable on a node with a taint and the NoSchedule effect, you can add tolerations to the pod config:

```
spec:
  containers:
  - name: grafana
    image: grafana
    imagePullPolicy: IfNotPresent
  nodeSelector:
    <label-key>: <label-value>
  tolerations:
  - key: <taint-key>
    operator: "Exists"
    effect: "NoSchedule"
```

You can also use taints to remove pods from a worker node. For example, if a pod is already running on a node, and you want to move it to another node, you can set a `taint` on the node with the `effect` of `NoExecute`, then pods without the matching `toleration` and `effect` will get moved out of the node. For example, you can add the following toleration to the above pod config to achieve this.

```
spec:
  containers:
  - name: grafana
    image: grafana
    imagePullPolicy: IfNotPresent
  nodeSelector:
    <label-key>: <label-value>
  tolerations:
  - key: <taint-key>
    operator: "Exists"
    effect: "NoSchedule"
  - key: <taint-key>
    `perator: "Exists"
    effect: "NoExecute"
```

