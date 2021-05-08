# Kubernetes Basics

## Overview

### Containers

<img align="right" src="https://github.com/kyminbb/ckad-prep/blob/main/basics/docs/images/container-structure.png" width="30%" height="30%">

- Completely isolated environment
- Different containers share the same kernel -> only containerize applications
- Instantiation of Docker Images, which easily resolve dependencies independently of the deployment environment
- Container orchestration
  - Process of automatically deploying and managing containers
  - ex) Docker Swarm, Kubernetes, Mesos

### Kubernetes Architecture

- Node
  - Worker machine where containers belong
  - Can be physical or virtual
  - We need more than one node in case of failure
- Cluster
  - A set of nodes
  - Multiple nodes enable applications to be accessible even when some of them fail, and distribute loads
- Master
  - Responsible of actual orchestration of nodes
- Components
  - API server
    - Frontend of Kubernetes
    - Provides APIs to interact with cluster
  - etcd
    - Distributed, reliable key-value store for information to manage cluster
  - Scheduler
    - Distributes work or containers across multiple nodes
  - Controller
    - Notices and responds when nodes go down
    - Determines to bring up new containers if necessary
  - Container runtime
    - Underlined software to run containers
    - ex) Docker
  - kubelet
    - Agent that runs on each node

<p align="center">
  <img src="https://github.com/kyminbb/ckad-prep/blob/main/basics/docs/images/master-worker-nodes.png" width="70%" height="70%">
</p>

## Kubernetes Concepts

### Pod

- A single instance of an application
- The smallest unit you can create in Kubernetes object model
- Encapsulates a container
- Sometimes a pod can consist of multiple containers, yet of different applications
- Pod definition

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: <pod_name>
    labels:
      [key_value_pairs]
  spec:
    containers:
      - name: <container_name>
        image: <image>
      ...
  ```

  Create with `kubectl create -f <yaml_file>`
- Creating a pod

  ```bash
  kubectl run <pod_name> --image=<image_name>
  ```

- List of pods available

  ```bash
  kubectl get pods [-o wide]
  ```

- Pod information

  ```bash
  kubectl describe pod <pod_name>
  ```

### Replica set

- Replica set definition
  
  ```yaml
  apiVersion: apps/v1
  kind: ReplicaSet
  metadata:
    name: <replica_set_name>
    labels:
      [key_value_pairs]
  spec:
    template:
      <pod_definition>
    replicas: <num_replicas>
    selector: 
      matchLabels:
        [key_value_pairs_of_pods_to_manage]
  ```

  Create with `kubectl create -f <yaml_file>`
- List of replica sets

  ```bash
  kubectl get replicaset
  ```

- Deleting replica set

  ```bash
  kubectl delete replicaset <replica_set_name>
  ```

  - Also deletes all underlying pods
- Updating replica set spec
  
  ```bash
  kubectl replace -f <yaml_file>
  ```

  ```bash
  kubectl scale --replicas=<new_num_replicas> replicaset <replica_set_name>
  ```

### Deployment

- Manages rolling updates of the underlying instances and applies changes
- Deployment strategy
  - Recreate
    - Apply changes to all pods altogether
    - Application goes down and becomes inaccessible to users
  - Rolling update
    - Apply changes to pods one by one
    - Default strategy of Kubernetes deployment
- Deployment definition

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: <deployment_name>
    labels:
      [key_value_pairs]
  spec:
    template:
      <pod_definition>
    replicas: <num_replicas>
    selector: 
      matchLabels:
        [key_value_pairs_of_pods_to_manage]
  ```

  Create with `kubectl create -f <yaml_file>`
- Updating deployment

  ```bash
  kubectl apply -f <yaml_file>
  ```

- Deleting deployment

  ```bash
  kubectl delete deployment <deployment_name>
  ```

- Rollout

  ```bash
  kubectl rollout status <deployment_name>
  ```

  - Viewing rollout history

    ```bash
    kubectl rollout history <deployment_name>
    ```

  - Rollback

    ```bash
    kubectl rollout undo <deployment_name>
    ```

## Networking in Kubernetes

- IP address is assigned to each pod
- Cluster networking
  - All containers/pods must communicate to one another without NAT (Network Address Translation)
  - All nodes must communicate with all containers and vice-versa without NAT

## Services

### NodePort

- Enables applications to be accesible to users
- Listens to a port on a node and forwards requests on that port to inner pods

<p align="center">
  <img src="https://github.com/kyminbb/ckad-prep/blob/main/basics/docs/images/nodeport.png" width="70%" height="70%">
</p>

- NodePort definition
  
  ```yaml
  apiVersion: v1
  kind: Serivce
  metadata:
    name: <service_name>
  spec:
    type: NodePort
    ports:
      - targetPort: [target_port]
        port: <port>
        nodePort: [node_port]
    selector:
      [key_value_pairs_of_pods_to_manage]
  ```
  
  Create with `kubectl create -f <yaml_file>`

### ClusterIP

- Allows communication between different microservices
- ClusterIP definition

  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: <service_name>
  spec:
    type: ClusterIP
    ports:
      - targetPort: [container_port]
        port: <service_port>
    selector:
      [key_value_pairs_of_pods_to_manage]
  ```
  
  Create with `kubectl create -f <yaml_file>`

### Load Balancer

- Acts as a reverse proxy
- Distributes network or application traffic across clusters
