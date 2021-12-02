# Project Report

## Project Goal

## What is Vertical Pod AutoScaler (VPA)?

## Tech Stack for this Demo
- RedHat OpenShift Cluster Platform (OCP)
  -  Note: VPA is provided by Kubernetes, so you can work with VPA with any other cloud platform, but we have used OCP and hence the instructions are provided for that here.
- Resource Consumer application
  -  Resource Consumer is an application tool to to generate the CPU/Memory workload inside a container.
  -  It starts the HTTP server when deployed, listens to requests at default port 8080 and handles those reqests.
  -  We use this application to test the vertical pod autoscaling by sending various HTTP load requests to resource consumer. 
 

## Steps to setup VPA in your namespace

### Prerequisites
We are assuming that you already have the following:
- a RedHat OpenShift Cluster Platform (OCP) and namespace ready to work on. 
- admin access in your cluster (This is required to install VPA operator in your cluster)

### Install VPA operator in your cluster 
1. In OCP, click on "Operators" -> "Operator Hub"
2. Choose " VerticalPodAutoscaler" and click Install
3. On the Install Operator page, ensure that the Operator recommended namespace option is selected. This installs the Operator in the mandatory openshift-vertical-pod-autoscaler namespace, which is automatically created if it does not exist.
4. Click Install.
5. Verify the installation by listing the VPA Operator components:
  1. Go to Workloads → Pods.
  2. Select the openshift-vertical-pod-autoscaler project from the drop-down menu and verify that there are four pods running.
  3. Navigate to Workloads → Deployments to verify that there are four deployments running.
6. [Optional] Verify the installation in the OCP CLI using the following command:
`$ oc get all -n openshift-vertical-pod-autoscaler`



### Inital Setup in your namespace

#### Create a deployment
1. Go to "Workloads" in your namespace under "Administrator" tab in moc/smaug.
2. Click on "Deployments" on the left side navigation bar.
3. Click on "Create Deployments" button which will open the yaml file. 
4. The yaml file will have preloaded entries, delete all of them.
5. Now, [copy and paste this exact yaml entries](https://github.com/TheGreymanShow/vertical-pod-autoscaler-operator/blob/main/install/rc-depl.yaml).
6. [Optional] you can change the name of the deployment, containers and app.
7. Click on Create

#### Create a service
1. Go to "Networking" in your namespace under "Administrator" tab in moc/smaug.
2. Click on "Services" on the left side navigation bar.
3. Click on "Create Service" button which will open the yaml file.
4. The yaml file will have preloaded entries, delete all of them.
5. Now, [copy and paste this exact yaml entries](https://github.com/TheGreymanShow/vertical-pod-autoscaler-operator/blob/main/install/rc-service.yaml).
6. Verify the "app" entry matches the deployment of app you created.
7. [Optional] you can change the name of the service.
8. Click on Create

#### Create a route 
1. Go to "Networking" in your namespace under "Administrator" tab in moc/smaug.
2. Click on "Routes" on the left side navigation bar.
3. Click on "Create Route" button which will open the yaml file.
4. The yaml file will have preloaded entries, delete all of them.
5. Now, [copy and paste this exact yaml entries](https://github.com/TheGreymanShow/vertical-pod-autoscaler-operator/blob/main/install/rc-route.yaml).
6. Verify the "name" entry of the service under spec: matches the name of the service you created.
7. [optional] you can change the name of the service.
8. Click on Create

### Create VPA Custom Resource in your namespace 

### Create VPA Controller


## How to see VPA recommendation? 

## How to check VPA changing the pod's metrics in Initial mode?

## How to check VPA changing the pod's metrics in Auto mode?

