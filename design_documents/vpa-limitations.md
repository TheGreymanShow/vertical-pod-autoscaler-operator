# Limitations and Advantages of VPA

This document explains the benefits of using VPA for autoscaling a Kubernetes cluster as well as the Limitations to keep in mind before using it.

## Benefits

- Cluster nodes are used efficiently because Pods use exactly what they need.

- Pods are scheduled onto nodes that have the appropriate resources available.

- You don't have to run time-consuming benchmarking tasks to determine the correct values for CPU and memory requests.

- Maintenance time is reduced because the autoscaler can adjust CPU and memory requests over time without any action on your part.


## Limitations

- Vertical Pod autoscaling does not generate recommendations based on sudden increases in resource usage. Instead, it provides stable recommendations over a longer time period. For sudden increases, use the Horizontal Pod Autoscaler.

- Vertical Pod autoscaling supports a maximum of 500 VerticalPodAutoscaler objects per cluster.

- Vertical Pod autoscaling is not yet ready for use with JVM-based workloads due to limited visibility into actual memory usage of the workload.

- Updating running pods is an experimental feature of VPA. Whenever VPA updates the pod resources the pod is recreated, which causes all running containers to be restarted. The pod may be recreated on a different node.

- VPA does not evict pods which are not run under a controller. For such pods Auto mode is currently equivalent to Initial.

- Vertical Pod Autoscaler should not be used with the Horizontal Pod Autoscaler (HPA) on CPU or memory at this moment. However, you can use VPA with HPA on custom and external metrics. To use vertical Pod autoscaling with horizontal Pod autoscaling, use multidimensional Pod autoscaling. You can also use vertical Pod autoscaling with horizontal Pod autoscaling on custom and external metrics.

- The VPA admission controller is an admission webhook. If you add other admission webhooks to you cluster, it is important to analyze how they interact and whether they may conflict with each other. The order of admission controllers is defined by a flag on APIserver.

- VPA reacts to most out-of-memory events, but not in all situations.

- VPA performance has not been tested in large clusters.

- VPA recommendation might exceed available resources (e.g. Node size, available size, available quota) and cause pods to go pending. This can be partly addressed by using VPA together with Cluster Autoscaler. Multiple VPA resources matching the same pod have undefined behavior. 

- It requires at least two healthy pod replicas to work. This kind of defeats its purpose in the first place and is the reason why it isn’t used extensively. As a VPA destroys a pod and recreates it to vertically autoscale it, it requires at least two healthy pod replicas to ensure there’s no service outage. That creates unnecessary design complications on single instance stateful applications, and you will have to look at a replicated design. For stateless applications, you can be better off using a Horizontal Pod Autoscalar instead of VPA.

- Minimum memory allocation is 250MiB by default. VPA allocates a minimum memory of 250MiB regardless of what you specify. Though this default can be modified at a global level, it makes it impractical for applications that consume less memory.

- It cannot be used with individual pods. VPA only works with Deployments, StatefulSets, DaemonSets, ReplicaSets etc. You cannot use it with a standalone Pod that does not have an owner. It isn’t a significant limitation because most of us don’t use standalone Pod anyway, but its worth mentioning.



