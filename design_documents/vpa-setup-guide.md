# VPA Setup Guide

This document introduces the VPA operator for Kubernetes, how it works and how you can set it up to use it for your cluster.

## Introduction

In Kubernetes, while actual workloads run within containers, Pods are in fact the smallest deployable unit of computing. This means that all compute resources (CPU/Memory) for any workload are specified at the Pod level. In Kubernetes, both forms of compute resources, CPU and Memory, are specified in similar ways and are done so in a static manner. When managing workloads via Pod, a human operator will first specify what compute resources this Pod will require to operate. It is not uncommon, however, for a workload to demand more resources at peak times, while remaining mostly dormant during off-peak hours. As one might expect, this results in either waste of compute resources or reduced pod performance.

The solution here is to move away from manually specifying the compute resources for a pod that are static, and opt for a more automated scaling solution. The ideal solution would identify when a Pod requires additional resources, and adjust its requirements accordingly during the lifetime of the Pod. The Vertical Pod Autoscaler (VPA) was designed to specifically do this. The VPA will automatically scale the resources for a pod based on its usage trends.

## How does VPA work

The Vertical Pod Autoscaler Operator (VPA) is implemented as an API resource and a custom resource (CR). The CR determines the actions the Vertical Pod Autoscaler Operator should take with the pods associated with a specific workload object, such as a daemon set, replication controller, and so forth, in a project.

The VPA automatically computes historic and current CPU and memory usage for the containers in those pods and uses this data to determine optimized resource limits and requests to ensure that these pods are operating efficiently at all times. For example, the VPA reduces resources for pods that are requesting more resources than they are using and increases resources for pods that are not requesting enough.

The VPA automatically deletes any pods that are out of alignment with its recommendations one at a time, so that your applications can continue to serve requests with no downtime. The workload objects then re-deploy the pods with the original resource limits and requests. The VPA uses a mutating admission webhook to update the pods with optimized resource limits and requests before the pods are admitted to a node. If you do not want the VPA to delete pods, you can view the VPA resource limits and requests and manually update the pods as needed.

For example, if you have a pod that uses 50% of the CPU but only requests 10%, the VPA determines that the pod is consuming more CPU than requested and deletes the pod. The workload object, such as replica set, restarts the pods and the VPA updates the new pod with its recommended resources.

For developers, you can use the VPA to help ensure your pods stay up during periods of high demand by scheduling pods onto nodes that have appropriate resources for each pod.

Administrators can use the VPA to better utilize cluster resources, such as preventing pods from reserving more CPU resources than needed. The VPA monitors the resources that workloads are actually using and adjusts the resource requirements so capacity is available to other workloads. The VPA also maintains the ratios between limits and requests that are specified in initial container configuration.

## Is VPA suitable for your service?

Despite the obvious motivation for using VPA, it is not a universally applicable solution...

