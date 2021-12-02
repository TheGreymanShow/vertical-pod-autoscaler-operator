# Project Report

## Project Goal

## What is Vertical Pod AutoScaler (VPA)?


## Steps to setup VPA in your namespace

### Prerequisites
We are assuming that you already have the following:
- a RedHat OpenShiftCluster (OPC) and namespace ready to work on. 
- admin access in your cluster (This is required to install VPA operator in your cluster)

Note: VPA is provided by Kubernetes, so you can work with VPA with any other cloud platform, but we have used OPC and hence the instructions are provided for that here.

### Inital Setup
In our project we used "resource-consumer" which is a tool to to generate the CPU/Memory workload inside a container. 

#### Create a deployment

#### Create a service

#### Create a route 

### Install VPA operator in your cluster 

### Create VPA Custom Resource in your namespace 

### Create VPA Controller


## How to see VPA recommendation? 

## How to check VPA changing the pod's metrics in Initial mode?

## How to check VPA changing the pod's metrics in Auto mode?

