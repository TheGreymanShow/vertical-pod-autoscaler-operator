# Limitations and Advantages of VPA

This document explains the benefits of using VPA for autoscaling a Kubernetes cluster as well as the Limitations to keep in mind before using it.

## Advantages
* Effective resource utilization: Since VPA auto-scales the resources dynamically, it helps in reducing consumption when they would be either otherwise. On the other end, if services need more resources due to heavy workload, instead of going out of resources and being unable to serve users, autoscaling can help here.
* VPA handles both upscaling and downscaling of resources, without any manual intervention.
* Finding the right values of request and limits for resources is hard. Even if you dont want to use autoscaling, using VPA in `Off` update policy, i.e recommendation only mode will help in figuring out the suitable values of resources for each application.
* VPA removes static definition of resources, which could be wrong and enables us to opt for automated solution. 
* With experimental feature of updating running pods, deployment/service of applications will not be interrupted.


## VPA Limitations
* VPA adjusts the CPU and memory allocation values of a pod by deleting and re-created new pod with updated configs. Even though rolling deployment can be configured, this is not desirable for some applications as it restarts the pods. Updating running pods is an experimental feature of VPA.
* VPA does not instantly react to varying workload. Instead, it provides stable recommendations over a longer time period. For sudden increases, Horizontal Pod Autoscaler is a better option.
* VPA metrics are not exposed from kube-state-metrics by default, as its a custom resource. We have to pass a flag to enable VPA collector. [Steps to do this.](https://github.com/kubernetes/kube-state-metrics/blob/master/docs/verticalpodautoscaler-metrics.md#configuration)
* VPA recommendation might exceed available resources (e.g. Node size, available size, available quota) and cause pods to go pending. Lets say your node has only 8gb memory, but VPA can actually recommend 10gb, as it does not consider available resource quota. So we have to manually set boundaries to avoid problems. We have solved this problem by setting `minAllowed` and `maxAllowed` [config](https://github.com/kubernetes/autoscaler/blob/master/vertical-pod-autoscaler/pkg/apis/autoscaling.k8s.io/v1/types.go#L153) values to define boundaries for VPA.
* Vertical Pod autoscaling supports a maximum of 500 VerticalPodAutoscaler objects per cluster.
* VPA does not evict pods which are not run under a controller. For such pods Auto mode is currently equivalent to Initial.
* Vertical Pod Autoscaler should not be used with the Horizontal Pod Autoscaler (HPA) on CPU or memory at this moment. However, VPA can be used with HPA on custom and external metrics.
* The VPA admission controller is an admission webhook. If you add other admission webhooks to you cluster, it is important to analyze how they interact and whether they may conflict with each other. The order of admission controllers is defined by a flag on APIserver.
* VPA performance has not been tested in large clusters.
* Multiple VPA resources matching the same pod have undefined behavior.
