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


## How to setup VPA Operator

Now that you have identified that your service is a suitable candidate for VPA, let's see how we can install and setup VPA on your Cluster.

Cluster administrators can install Operators to an OpenShift Container Platform cluster by subscribing Operators to namespaces with OperatorHub.

### VPA installation with OperatorHub

OperatorHub is a user interface for discovering Operators; it works in conjunction with Operator Lifecycle Manager (OLM), which installs and manages Operators on a cluster.

As a cluster administrator, you can install an Operator from OperatorHub using the OpenShift Container Platform web console or CLI. Subscribing an Operator to one or more namespaces makes the Operator available to developers on your cluster.

You can install and subscribe to an Operator from OperatorHub using the OpenShift Container Platform web console.

### Prerequisites
- Access to an OpenShift Container Platform cluster using an account with cluster-admin permissions.

### Procedure

1. Navigate in the web console to the Operators → OperatorHub page.

2. Scroll or type a keyword into the Filter by keyword box to find the Operator you want. For example, type advanced to find the Advanced Cluster Management for Kubernetes Operator.

   You can also filter options by Infrastructure Features. For example, select Disconnected if you want to see Operators that work in disconnected environments, also known as restricted network environments.

3. Select the Operator to display additional information.

```Note: Choosing a Community Operator warns that Red Hat does not certify Community Operators; you must acknowledge the warning before continuing.```

4. Read the information about the Operator and click Install.

5. On the Install Operator page:

    - Select one of the following:

      - All namespaces on the cluster (default) installs the Operator in the default openshift-operators namespace to watch and be made available to all namespaces in the cluster. This option is not always available.

      - A specific namespace on the cluster allows you to choose a specific, single namespace in which to install the Operator. The Operator will only watch and be made available for use in this single namespace.

    - Select an Update Channel (if more than one is available).

    - Select Automatic or Manual approval strategy, as described earlier.

6. Click Install to make the Operator available to the selected namespaces on this OpenShift Container Platform cluster.

    - If you selected a Manual approval strategy, the upgrade status of the subscription remains Upgrading until you review and approve the install plan
  
    - After approving on the Install Plan page, the subscription upgrade status moves to Up to date.

    - If you selected an Automatic approval strategy, the upgrade status should resolve to Up to date without intervention.

7. After the upgrade status of the subscription is Up to date, select Operators → Installed Operators to verify that the cluster service version (CSV) of the installed Operator eventually shows up. The Status should ultimately resolve to InstallSucceeded in the relevant namespace.