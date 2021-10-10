# Choosing candidate for VPA testing

## Goals

### Option 1: Utilizing Existing Load Testing Tools

In this option, our aim is to utilize the existing load testing tools to test
the VPA. To achieve this, we could deploy a web server to our pod
which we can send request via a load testing tool and generate artificial traffic.
By this way, with each request we send, we are expecting cpu and memory usage of
the pod to be increased hence we will indirectly control the pod resources.

We went through the existing load testing tools including Speedscale, K6,
Apache JMeter, Gatling and ReadyAPI. Among those options, K6 is the most complete tool
which is very easy to install, and they have extensive documents describing 
how to run the load test on a kubernetes cluster.

How to install K6:
https://k6.io/docs/getting-started/installation/

How to run local test:
https://k6.io/docs/getting-started/running-k6/

How to run distributed K6 tests on K8s cluster:
https://k6.io/blog/running-distributed-tests-on-k8s/

### Option 2

### Option 3: K8s Resource Consumer
Credits/More info: https://github.com/kubernetes/kubernetes/tree/master/test/images/resource-consumer

#### Introduction
Resource Consumer is primarily developed to test k8s autoscaling. It helps to test cluster size autoscaling, Horizontal Pod Autoscaler(HPA), and Vertical Pod Autoscaler(VPA) operators in Kubernetes. This tool allows to generate CPU/Memory consumption in a container by starting an HTTP server and creates a new process for every POST request.

The container consumes requested amount of resources and are specified as follows:
- CPU in millicores
- Memory in megabytes

#### Install Resource Consumer
```
kubectl run resource-consumer --image=gcr.io/k8s-staging-e2e-test-images/resource-consumer:1.9 --expose --service-overrides='{ "spec": { "type": "LoadBalancer" } }' --port 8080 --requests='cpu=500m,memory=256Mi'
kubectl get services resource-consumer
```
or by creating a ```deployment.yaml``` file and specifying ```spec.template.spec.containers[].image: gcr.io/k8s-staging-e2e-test-images/resource-consumer:1.9``` and deploying it using ```kubectl apply -f <filename.yaml>```

#### To send HTTP request for CPU - 
- API: **/ConsumeCPU**
- params: millicores & durationSec
- Example command: curl --data "millicores=600&durationSec=100" http://<EXTERNAL_IP>/ConsumeCPU

#### To send HTTP request for Memory - 
- API: **/ConsumeMem**
- params: megabytes & durationSec
- Example command: curl --data "megabytes=200&durationSec=100" http://<EXTERNAL_IP>/ConsumeMem


## Comparison

| Criteria/Approach                  | Web Server + Load testing tools | Scripts | K8s Resource Consumer |
|------------------------------------|---------------------------------|---------|-----------------------|
| Ability to control CPU load        | It is not a trivial task to control the CPU utilization with server request.                               |         | Easy to control CPU consumption using simple curl command.                      |
| Ability to control Memory workload | It is easily configurable how much memory will be consumed with each request in web server.                                |         | Easy to control Memory consumption using simple curl command.                      |
| Ease of setup/development          | Easy to configure K6 but some effort is required to deploy a web server to pod. |         | Easy setup using existing image that includes backend HTTP server and controller logic to stress resources.                      |
| Compatibilty                                  |                                 |         | Primarily developed to test k8s autoscaling.                      |

## Preferred Approach and reason

