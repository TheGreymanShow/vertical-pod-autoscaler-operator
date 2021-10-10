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

### Option 3

## Comparison

| Criteria\Approach                  | Web Server + Load testing tools | Scripts | K8s Resource Consumer |
|------------------------------------|---------------------------------|---------|-----------------------|
| Ability to control CPU load        | It is not a trivial task to control the CPU utilization with server request.                               |         |                       |
| Ability to control Memory workload | It is easily configurable how much memory will be consumed with each request in web server.                                |         |                       |
| Ease of setup/development          | Easy to configure K6 but some effort is required to deploy a web server to pod. |         |                       |
|                                    |                                 |         |                       |

## Preferred Approach and reason

