# Blog: Vertically Scale your application using the Vertical Pod Autoscaler Operator on Redhat Openshift 

## The Basics
Kubernetes is hailed for making it simple to provision the resources you need, when you need them. However, it’s challenging to know the exact size and number of nodes that best fit your application, especially when you can’t predict what load you will want to support in the future. If you are allocating resources manually, you may not be quick enough to respond to the changing needs of your application 

The solution here is to move away from manually specifying the compute resources for a pod that are static, and opt for a more automated scaling solution. The ideal solution would identify when a Pod requires additional resources, and adjust its requirements accordingly during the lifetime of the Pod. The Vertical Pod Autoscaler (VPA) was designed to specifically do this. The VPA will automatically scale the resources for a pod based on its usage trends.

This blog describes the work we did to test and deploy a Vertical Pod Autoscaler operator on Trino, a production workload running live in Redhat’s operate-first cluster. You will learn the entire process we did right from setting up the cluster and installing the VPA operator, testing it by exposing it to various real-life workload simulations and finally deploying it on a live running application.


## Background - Autoscaling in Kubernetes
Before we dive into understanding how VPA works, it would be helpful to take a step back and understand how autoscaling is handled in kubernetes and comparing the 3 main types of autoscaling - horizontal, vertical and cluster autoscaling. This will help set the context for the focus of this blog, which will be vertical autoscaling.

Kubernetes provides multiple layers of autoscaling functionality: the Horizontal Pod Autoscaler, the Vertical Pod Autoscaler, and the Cluster Autoscaler.  Together, these allow you to ensure that each pod and cluster is just the right size to meet your current needs.

### Horizontal Autoscaling
Horizontal autoscaling refers to the ability to scale wider as per the increasing traffic. It is the capability to amend several hardware or software entities such as servers so that they execute as a single logical unit. Such scaling cannot be implemented in short notice.

#### How Kubernetes handles it
- Kubernetes handles the problem of horizontal scaling in two layers, one is scaling horizontally by increasing the number of pods available in a cluster in response to the present computation needs. You specify the metrics that will determine the number of pods needed, and set the thresholds at which pods should be created or removed. It uses the Horizontal Pod Autoscaler to do so.
- Another way Kubernetes handles horizontal autoscaling is at the cluster level, by adding more nodes to your cluster based on the number of pending pods. It uses the Cluster Autoscaler to see whether there are any pending pods and increases the size of the cluster so that these pods can be created. The CA also deallocates idle nodes to keep the cluster at the optimal size.

### Vertical Autoscaling
Vertical Autoscaling simply refers to adding more power to an existing instance. It resizes the server with no modifications to the base code. It is the ability to enhance the capacity of the existing hardware or software by including additional resources.

#### How Kubernetes handles it
- Where the HPA allocates pod replicas in order to manage resources, the Vertical Pod Autoscaler (VPA) simply allocates more (or less) CPUs and memory to existing pods. This can be used just to initialize the resources given to each pod at creation, or to actively monitor and scale each pod’s resources over its lifetime. 

## Goals
Now that we have explored the different autoscaling solutions that kubernetes provides, let’s come back to the focus of this blog - Vertical Autoscaling. Our Goal with the project was to deploy a vertical pod autoscaler on a live production workload in RedHat’s operate-first org. But before deploying the VPA on the workload, we recognized that it was extremely important to test VPA on a sample application, simulate various real life workload simulations to observe whether VPA reacts as expected by increasing or decreasing the resources appropriately. We’ll discuss all the experiments we did with VPA in this blog before being confident of deploying it on a real production workload.
 
## Setup

The following is the tech stack that we used for our project.

- **Openshift Cluster:** For deploying a new kubernetes based cluster
- **VPA operator:** Operator installed for autoscaling the pods vertically
- **Github:** for storing all code and config files
- **Kube-state metrics:** for retrieving usage metrics from pods
- **Prometheus:** Storing these metrics in time-series format database
- **Grafana:** visualizing the real time metrics by creating a dashboard


We are assuming that you already have the following:

- a RedHat OpenShift Cluster Platform (OCP) and namespace ready to work on.
- admin access in your cluster (This is required to install VPA operator in your cluster)
- `kubectl` installed in your local machine
- `openshift command line interface` installed. Refer this link
- [Only for kubectl] login to your OCP in terminal (Use copy login command)

### Installing VPA
Openshift provides a very easy way to install operators on your cluster via the OperatorHub. You can follow this [link](https://docs.openshift.com/container-platform/4.9/nodes/pods/nodes-pods-vertical-autoscaler.html#nodes-pods-vertical-autoscaler-install_nodes-pods-vertical-autoscaler) for detailed steps on how to install the VPA operator on your cluster.


### How to use VPA

VPA introduces a couple of Custom Resource Definitions (CRD for short) to control automatic recommendations behavior. Typically Developers would add a VerticalPodAutoscaler object to their application deployments, and a VerticalPodAutoscalerController as a Controller for this Resource Definition. You need to define these two objects, in order to use VPA.

On your Openshift dashboard, as an administrator, 
- Go to Home -> API explorer
- Search for VerticalPodAusocaler
- Go to instances
- Click the `VerticalPodAutoscaler` button to define and new VPA object as follows.
- Copy/Paste this config yaml to setup your VPA object.

```
apiVersion: "autoscaling.k8s.io/v1beta2"
kind: VerticalPodAutoscaler
metadata:
  name: resource-consumer
  namespace: cs6620-fall21-deployverticalpod
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: rc-deployment
  updatePolicy:
    updateMode: "Auto"
```

Next, follow the same steps to create a VerticalPodAutoscalerController object instead.

VerticalPodAutoscaler Controller config.yaml:

```
apiVersion: autoscaling.openshift.io/v1
kind: VerticalPodAutoscalerController
metadata:
  name: resource-consumer
  namespace: cs6620-fall21-deployverticalpod
spec:
  minReplicas: 1
```



## Initial Testing

Once you have your VPA object and controller setup, we can now try to test it on any application deployed on the cluster and check how it reacts to some basic scenarios such as memory/cpu over and under utilization. 

Now you don’t need to break your head for selecting the sample application for testing, fortunately there already exists an application built for this exact purpose. Resource Consumer is a tool which allows to generate cpu/memory utilization in a container. The reason why it was created is testing Kubernetes autoscaling. You can find the source code for this tool here. Now follow the following 3 steps to create a deployment, service and a route for this Resource Consumer Application on your cluster.

#### Create a deployment

1. Go to "Workloads" in your namespace under "Administrator" tab in moc/smaug.
2. Click on "Deployments" on the left side navigation bar.
3. Click on "Create Deployments" button which will open the yaml file. 
4. The yaml file will have preloaded entries, delete all of them.
5. Now, [copy and paste this exact yaml entries](https://github.com/TheGreymanShow/vertical-pod-autoscaler-operator/blob/main/install/rc-depl.yaml).
6. [Optional] you can change the name of the deployment, containers and app.
7. Click on "Create"

#### Create a service

1. Go to "Networking" in your namespace under "Administrator" tab in moc/smaug.
2. Click on "Services" on the left side navigation bar.
3. Click on "Create Service" button which will open the yaml file.
4. The above yaml file will have preloaded entries, delete all of them.
5. Now, [copy and paste this exact yaml entries](https://github.com/TheGreymanShow/vertical-pod-autoscaler-operator/blob/main/install/rc-service.yaml).
6. Verify the "app" entry matches the deployment of app you created.
7. [Optional] you can change the name of the service.
8. Click on "Create"

#### Create a route 

1. Go to "Networking" in your namespace under "Administrator" tab in moc/smaug.
2. Click on "Routes" on the left side navigation bar.
3. Click on "Create Route" button which will open the yaml file.
4. The above yaml file will have preloaded entries, delete all of them.
5. Now, [copy and paste this exact yaml entries](https://github.com/TheGreymanShow/vertical-pod-autoscaler-operator/blob/main/install/rc-route.yaml).
6. Verify the "name" entry of the service under spec: matches the name of the service you created.
7. [optional] you can change the name of the route.
8. Click on "Create"

#### Note:
- We use this route to send requests to our resource consumer application deployed. 
- Once this deployment is running, it will constantly consume “x” CPU and “y” Memory depending on the values defined in your deployment yaml.
- You can verify this by checking the resource usage graphs on your Openshift dashboard.


### Observing VPA changes

Now once you observe the usage consumption in Openshift dashboard, you can try to change the workload (increase or decrease it) to check if VPA reacts accordingly.

To change the workload, hit the following curl command

``` curl --data "millicores=300&durationSec=600" http://<EXTERNAL-IP>:8080/ConsumeCPU ```

This request ensures that 300 millicores of CPU will be consumed for 600 seconds. 

To check VPA’s new recommendation values, 
- you can go to the VPA object yaml that you created and check the value assigned for `target-recommendation` key. 
- It should be somewhat close to 300 millicores depending on the initial value you set while deploying the rc app. 
- The more you experiment by changing the load using the above command, the recommendation values will change indicating that VPA is monitoring your application resource usage, and reacting accordingly.
 
 
### Key observations made after initial testing
- VPA reacts to Over & under utilization well
- VPA Recommendations are instant
- But updating pods with new recommendations is not instant

  
## Workload Simulation

`Goal: Observe which scenarios does VPA react well, which will help in selecting the ideal candidate in production to deploy VPA`

Explain different Workload simulation scenarios and how to set them up
- Memory over utilization
- Memory under utilization
- Day and Night 
 
 
## Limitations of VPA

- Whenever VPA updates the pod resources the pod is recreated, which causes all running containers to be restarted. Updating running pods is an experimental feature of VPA.
- VPA does not react instantly to sudden spikes in CPU/Memory utilization.
- VPA cannot be used in combination with HPA, which scales based on same metrics.
- VPA recommendations can exceed the available resource quota. 
- It may not be suitable for applications that consume very low memory, as we never observed VPA recommending less than 250mb of memory. (This default minimum recommendation can be overridden with a flag)
 
## Future Work

- Expose the VPA recommendations metrics using kube-state-metrics to query in PromQL. 
- Install VPA in Jupyter Lab namespace in OCP. Jupyter Lab namespace does not have deployments only has pods. VPA only works with Deployments, StatefulSets, DaemonSets, ReplicaSets etc. You cannot use it with a standalone Pod that does not have an owner. Hence, we can create a custom controller and associate it with pods and associate the created custom controller to VPA custom resource.


## Credits
We would like to thank our mentors Humair Khan and Anand Sanmukhani, who are members of the OpenShift Operate First teams who guided us on this, as well as professor Peter Desnoyrs, from Northeastern University for helping us pace our project over the fall 2021 semester.
