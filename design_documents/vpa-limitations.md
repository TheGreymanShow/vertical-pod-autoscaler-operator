# Limitations and Advantages of VPA

This document explains the benefits of using VPA for autoscaling a Kubernetes cluster as well as the Limitations to keep in mind before using it.

## Advantages

...

## VPA Limitations
1. It takes time VPA to be autoscaled which is not happen instantly and is costly in terms of time. VPA does not generate recommendations based on sudden increases in resource usage. Instead, it provides stable recommendations over a longer time period. For sudden increases, Horizontal Pod Autoscaler is a better option.
2. Lack of configuration. There is not a lot of option that we can configure VPA promptly. It requires going over specs in detail to understand how it handles the upper and lower bound for example. We have to have flag for configuration options that we need to look at the status.
3. Metrics are not being exported. It does not matter where we deployed the VPA, this is needs to be managed at the operator level. 
4. Default configuration of VPA is defined with 250mb memory limit. This configuration can be changed in VPA api resource by using minAllowed definition, however, the default value may not be suitable for small applications.
5. Vertical Pod autoscaling supports a maximum of 500 VerticalPodAutoscaler objects per cluster.
6. Vertical Pod autoscaling is not yet ready for use with JVM-based workloads due to limited visibility into actual memory usage of the workload.
7. Updating running pods is an experimental feature of VPA. Whenever VPA updates the pod resources the pod is recreated, which causes all running containers to be restarted. The pod may be recreated on a different node.
8. VPA does not evict pods which are not run under a controller. For such pods Auto mode is currently equivalent to Initial.
9. Vertical Pod Autoscaler should not be used with the Horizontal Pod Autoscaler (HPA) on CPU or memory at this moment. However, VPA can be used with HPA on custom and external metrics.
10. The VPA admission controller is an admission webhook. If you add other admission webhooks to you cluster, it is important to analyze how they interact and whether they may conflict with each other. The order of admission controllers is defined by a flag on APIserver.
11. VPA reacts to most out-of-memory events, but not in all situations.
12. VPA performance has not been tested in large clusters.
13. VPA recommendation might exceed available resources (e.g. Node size, available size, available quota) and cause pods to go pending. This can be partly addressed by using VPA together with Cluster Autoscaler.
14. Multiple VPA resources matching the same pod have undefined behavior.
