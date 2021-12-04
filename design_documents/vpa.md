# VPA

## Introduction

...

## How does VPA work

...

## Is VPA suitable for your service?

Despite the obvious motivation for using VPA, it is not a universally applicable solution. Vertical Pod Autoscaler is a great way to automate the upscaling and downscaling the resource requests and limits depending on the historical usage data of a pod. But there are some thoughts to take into consideration before choosing VPA as the right fit for your application. Now let us take a look at these considerations -

**1. Not recommended for too small workloads**
    - Whatever value you specify for minAllowed, VPA allocates a minimum memory of 250MiB. Even if your requests are consuming less, it will be automatically adjusted to fit this bound. So, it is not recommended to use VPA for very small applications.

**2. Not recommended to be used with individual pods**
    - VPA is not designed to work with standalone pods that do not have a parent, such as a Deployment, StatefulSets, DaemonSets, ReplicaSets, etc.., It can work with any custom Controller object that can scale its subresource. 

**3. Not recommended for workloads that dont tolerate evictions or disruptions**
    - As VPA evicts and recreates a pod to update the resource recommendations, it requires at least two healthy pod replicas to make sure that there’s no service outage. For this reason, if we use single pod replica, there is a chance for application crashes so the developer should be reponsible to check that their app is up and running. If your application is not highly available and cannot stand with disruptions, use the INITIAL or OFF mode as the Update Policy so that VPA will not evict pods but only provides you with the recommendations based on the historical usage.
    
**4. Good for Stateless applications**
    - Stateless workloads can withstand evictions and sudden disruptions. So, in these applications, we can use the Auto or Recreate mode as the Update Policy. 

VPA is a good choice for applications with predictable resource usage and in applications where it is evident that increasing more instances or pods will not improve the efficiency. For these applications, it may not improve performance when using Horizontal Pod Autoscaling. Hence we can use VPA here.
   
