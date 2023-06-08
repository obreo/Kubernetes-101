# Kubernetes-101
Learn the basics of Kubernetes 


## Introduction
Kubernetes (K8) is an orchestration management tool, which is used to manage containers at a larger scale setup. It ensures high availability by scaling in and out the applications as much as it suits the project. 

## Basic Theory
K8 is mainly based on nodes, which are physical or virtual machines used to control or to setup the containers. There are mainly two node elements:
1. Master node: which is used as the controller and manager for the second element, the worker nodes.
2. Worker node: It hosts the containers required as one or multiple applications. Multiple nodes can be used for one or several purposes, each node may host one or multiple containers under a layer called Pod. Nodes can be replicated.

## Digging deeper

Note: All of the following components can be configured as script files or run directly by command line. unless some are running by default in the backend. The script file format used it YAML.

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
4. Statefulset: This component is used for stateful applications, which require persistent data storage, mostly used for databases.

#### Other Core components related to Worker node:
1. ConfigMap: It is a K8 resource that let's you configure your container application's data. Such as adding environment variables, setting keys.. etc.
2. Secrets: This is used to store your sensitive data and credintials for the application. By default, this data is unencypted, so you have to encrypt it yourself. You can use `echo -n YOUR_DATA_KEY | base64` to encrypt your values before storing them in the secrets file.
### Persistent Storage Type:
The data stored in K8 is ephemeral. This means it is temporary and can be lost once the once the pods are deleted. To solve this we can create a data storage similar to creating persistent storage in Docker containers, by setting a local volume to be synchronized with the K8 application.

## Installation & Configuration
### Setup
The easiest and most direct way to setup a kubernetes cluster is to use [Minikube](https://minikube.sigs.k8s.io) or using cloud services as AWS EKS and similar services in other cloud providers. This blog considers using minikube for its ease and its installation is demonstrated in the [documentation](https://minikube.sigs.k8s.io/docs/start/)

There are two main elements used to setup and configure Kubernetes:
1. Minikube: It creates the kubernetes cluster.
2. kubectl: this is the main way to interact with your nodes, retrieve information and configure them in kubernetes. It's installed by defailt with Minikube. 

### Configuration

As stated previously, each of the above components particularly in the worker node are prepared seperately by YAML script files, they will be linked together using labels. You can always check the [kubernetes documentation](https://kubernetes.io/docs/home/) for better understanding.

Minikube sets up a Kubernetes cluster with a single node that acts as both the master and worker node. So as the node is installed it prepares the master node configurations in the backend, you can check the cluster's status by:

```
minikube status
```

to check the node's status, use kubectl:

```
kubectl get nodes

# OR, for mode details

kubectl get node -o wide
```

for more options about the cluster, check the help:
```
minikube -h
```

You can use other commands for info about your deployments, pods, IP server, start, delete, strop and many other configurations using the help command:

```
kubectl -h
```

### Write your first application
This is a demo based on this [tutorial](https://www.youtube.com/watch?v=s_o8dwzRlu4) which will be demonstrated in detail:

We'll create an application in two seperate pods, where one pod will contain a web application and the other will contain a databse container.

#### Step #1 - Create the ConfigMap for the web app container:

This is used to set the static IP endpoint from the service to the configMap which will be used as the main URL for the database by the web app:
`mongo-configmap.yml `

```
apiVersion: v1
kind: ConfigMap                 # The type of script, there are different ones as pod, deployment, secret, service and more.
metadata:
  name: mongo-app               # labeling is important to refer to the script in other configuration components.
data:                           # Here is where you can configure the data you want for your app using key - value pairs.
  mongo-url: mongo-service      # This will represent the service to be used as endpoint
```
#### Step #2 - Create a Secrets file in which we will store the username and password data:
`mongo-secret.yml`

```
apiVersion: v1
kind: Secret                    # The type of element
metadata:
  name: mongo-secret            # This label will be used as a reference
type: Opaque                    # Default type for storing secret data
data:
  username: YWRtaW4=            # Your data input.
  password: cGFzc3dvcmQ=

```

#### Step #3 - Create a deployment file for the database pod and a service as a static IP to link it with our deployment:

```
# Deployment of Mongo databse:

apiVersion: apps/v1           # Deployment configuration
kind: Deployment                #Type - Deployment
metadata:
  name: mongo-deployment        # Label for the Deployment script
  labels:                       # This is a label that represents the depolyment configuration to be matched with the pod configuration below
    app: mongo
spec:
  replicas: 1                   # Replicas of the pods, because it's a database, one replica is recommended, otherwise use stateful deployment.
  selector:                     # This will match the pod's app label with the deployment's app label
    matchLabels:
      app: mongo
  template:                   # Pod configuration
    metadata:
      labels:                   # This is a label that represents the pod's configuration to be matched with the deployment configuration above
        app: mongo
    spec:
      containers:               # Containers to be installed on the pod
      - name: mongodb             # docker container's name
        image: mongo:6.0          # docker image - as stated in the dockerhub repository
        ports:                    # Port assigned to the container
        - containerPort: 27017
        env:                      # Environmment variables related to the image
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: username
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: password


---
# service configuration

apiVersion: v1
kind: Service
metadata:
  name: mongo-service   # name of service, and can be used similar to the data element in configmap to connect the pod with it.
spec:
  selector:             # Matching the label name of selector in the mongo deployment yml.
    app: mongo
  ports:
    - protocol: TCP
      port: 27017          # Listener port
      targetPort: 27017  # forward port: the one mentioned in the deployment file for container
```
