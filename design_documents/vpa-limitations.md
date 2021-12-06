# Limitations and Advantages of VPA

This document explains the benefits of using VPA for autoscaling a Kubernetes cluster as well as the Limitations to keep in mind before using it.

## Advantages

...

## VPA Limitations
1. VPA does not instantly react to varying workload. Instead, it provides stable recommendations over a longer time period. For sudden increases, Horizontal Pod Autoscaler is a better option.
2. Lack of configuration. There is not a lot of option that we can configure VPA promptly. It requires going over specs in detail to understand how it handles the upper and lower bound for example. We have to have flag for configuration options that we need to look at the status.
3. VPA metrics are not exposed from kube-state-metrics by default, as its a custom resource. We have to pass a flag to enable VPA collector. [Steps to do this.](https://github.com/kubernetes/kube-state-metrics/blob/master/docs/verticalpodautoscaler-metrics.md#configuration)
4. Default configuration of VPA is defined with 250mb memory limit. This configuration can be changed in VPA api resource by using minAllowed definition, however, the default value may not be suitable for small applications.
5. Vertical Pod autoscaling supports a maximum of 500 VerticalPodAutoscaler objects per cluster.
6. Vertical Pod autoscaling is not yet ready for use with JVM-based workloads due to limited visibility into actual memory usage of the workload.
7. VPA adjusts the CPU and memory allocation values of a pod by deleting and re-created new pod with updated configs. Even though rolling deployment can be configured, this is not desirable for some applications as it restarts the pods. Updating running pods is an experimental feature of VPA.
8. VPA does not evict pods which are not run under a controller. For such pods Auto mode is currently equivalent to Initial.
9. Vertical Pod Autoscaler should not be used with the Horizontal Pod Autoscaler (HPA) on CPU or memory at this moment. However, VPA can be used with HPA on custom and external metrics.
10. The VPA admission controller is an admission webhook. If you add other admission webhooks to you cluster, it is important to analyze how they interact and whether they may conflict with each other. The order of admission controllers is defined by a flag on APIserver.
11. VPA reacts to most out-of-memory events, but not in all situations.
12. VPA performance has not been tested in large clusters.
13. VPA recommendation might exceed available resources (e.g. Node size, available size, available quota) and cause pods to go pending. Lets say your node has only 8gb memory, but VPA can actually recommend 10gb, as it does not consider available resource quota. So we have to manually set boundaries to avoid problems. We have solved this problem by setting `minAllowed` and `maxAllowed` [config](https://github.com/kubernetes/autoscaler/blob/master/vertical-pod-autoscaler/pkg/apis/autoscaling.k8s.io/v1/types.go#L153) values to define boundaries for VPA.
14. Multiple VPA resources matching the same pod have undefined behavior.
