# Kubernetes-101
Learn the basics of Kubernetes 
This tutorial is based on this video:

https://www.youtube.com/watch?v=s_o8dwzRlu4

## Introduction
Kubernetes (K8) is an orchestration management tool, which is used to manage containers at a larger scale setup. It ensures high availability by scaling in and out the applications as much as it suits the project. 

## Basic Theory
K8 is mainly based on nodes, which are physical or virtual machines used to control or to setup the containers. There are mainly two node elements:
1. Master node: which is used as the controller and manager for the second element, the worker nodes.
2. Worker node: It hosts the containers required as one or multiple applications. Multiple nodes can be used for one or several purposes, each node may host one or multiple containers under a layer called Pod. Nodes can be replicated.

## Digging deeper

Within these two elements, there are a number of components that work for each of them and understanding their purpose is essencial to use Kubernetes:
### Master node components:
1. API server: It's the endpoint of your K8 cluster installed on the machine. This is used to validate your resources, also recieving, processing, and creating new resources. And can assure a highly availablity by deploying multiple API server instances as replicas in case of failure.
2. Scheduler: It acts as a load balancer, that it's used to assure scheduling a pod - will be discussed later - in the worker node specified to the desired number for the project while assuring the distribution between the worker nodes for a better utilization.
3. Controller manager: It's a service used in the backend to manage and monitor the changes automatically of the Kubernetes resources. It ensures fault tolerance, scalability and reliability of the applications.
4. Etcd: It saves and configures the key-value data used to configure the kubernetes cluster. The data stored and retrieved by the K8 configuration files use etcd as a source. Such as service definitions, Deployments, ConfigMaps, Secrets which will be discussed later in the article.

### Worker node components:
1. Pod: It's the smallest element in a node and used as a layer for a container or group of containers, A single pod represents an application in the node. Each pod has an IP that provides connectivity to the pod and communication between containers assigned in the Pod or with other Pods. However, if a Pod is rescheduled or replaced then its IP will change. To solve this issue we can provide another component that connect to the pod or deployment - will be discussed below - called a Service.
2. Service: A static IP endpoint for a pod or a number of pods that can be accessed inernally and externally. This helps the application to retain the ip endpoint even if a replacement or rescheduling occured to the pods. This helps pods - together - and clients to communicate without connectivity disturbance.
3. Deployment: Another layer component used over the pod's layer, which is used to configure the pod with more flexibility, particularly to increase the high availability of the pods by creating replicas within the node or multiple nodes - using the Scheduler.

#### Other Core components related to Worker node:
1. ConfigMap: 
2. Secrets:

### Persistent Storage Types:
