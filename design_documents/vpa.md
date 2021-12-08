# VPA

## Vertical Pod Autoscaler

Vertical Pod Autoscaler automatically updates the limits and requests of resources (both CPU and memory) by reviewing the historic resource usages and as well their current. 

#### VPA modes 
VPA can be configured in three different modes. They are
1. Off - VPA does not change the pod's request and limit. Just gives the recommendations for the containers.
2. Initial - VPA applies the recommendations only during the pod's creation. 
3. Auto - VPA automatically applies the recommendations when the resource usage goes beyond the lowerbound and upperbound.

#### VPA Architecture

![VPA Architecture](images/vpa_architecture.png)

VPA Architecture [Image Source](https://banzaicloud.com/blog/k8s-vertical-pod-autoscaler/)


VPA has three importent components. 
1. VPA recommender
2. VPA admission controller
3. VPA Updater

##### VPA Recommender 
VPA Recommender continously monitors the pod's resource levels (in our project, it gets the pod's resource levels using kube state metrics) and provides the recommendation values It also fills the recommendations in the VPA resource's output only field. 

These recommendations will have the following four fields:
1. LowerBound - minimum resource levels recommended
2. Target - the recommended resource level
3. Uncapped Target - the recommended resource levels, without considering the `minAllowed` and `maxAllowed` restrictions in VPA
4. UpperBound - maximum resource levels recommended

##### VPA Admission Controller
VPA admission controller acts only during two modes. `Auto` and `Initial`. It intercepts the pod's creation and updates their requests and limits with the recommended resources. 
VPA admission controller calls the VPA recommender to get the recommended resources. It works in both pod's creation and recreation. 

##### VPA Updater
VPA Updater acts only during the `Auto` mode. VPA Updater is responsible for updating the resource levels in the running pod. 

##### How and when does VPA updates the live running pod?
VPA Updater continously montiors the resource levels and gets the recommendations from the VPA recommender. If the pod's resource requests, goes below the lowerbound or goes above the upperbound, VPA updater evicts the pod. The pod recreation will not be initiated by any of the VPA components. For this, VPA depends on the deployment's replicaset, resource policy and so on. 
During the pod's recreation, it is the admission controller's job to apply the recommended resources. 

#### Logs Observations

#### Log level
Once the VPA operator is installed (refer this on how to install), it creates a default pod for each components. Initial log level for these VPA pods will be 1. You need to change the log level to atleast 4 to see the vpa logs such as metrics changed in Auto mode, creation mode and also the recommendations.

#### Logs 
##### VPA Recommender logs
It will log when the recommendations have been changed.

##### VPA Admission Controller logs
It will log when the pod's resource recommendations are applied to the pod's creation/recreation. 

##### VPA Updater logs
It will log why the pod needs to be updates, and also logs why the pod need not be updated because they are well within the upper and lower limits. 


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
   
