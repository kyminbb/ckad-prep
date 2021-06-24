# ckad-prep

## Configuration

### Commands and Arguments

- Overwrite commands and arguments of container that runs in pod

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: <pod_name>
  spec:
    containers:
    - name: <container_name>
      image: <image_name>
      command: [command_list]    # ["sleep"]
      args: [argument_list]    # ["5000"] 
  ```
  
  - `command` replaces `ENTRYPOINT` of dockerfile
  - `args` replaces `CMD` of dockerfile

### Config map

- Config map definition

  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: <config_map_name>
  data:
    <key_value_pairs>
  ```

  Create with `kubectl create -f <yaml_file>`
- Creating config map

  ```bash
  kubectl create configmap <config_name> --from-literal=<key>=<value>
  ```

  ```bash
  kubectl create configmap <config_name> --from-file=<config_file>
  ```

- Injecting config map into pod

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: <pod_name>
  spec:
    containers:
    - name: <container_name>
      image: <image>
      envFrom:
      - configMapRef:
          name: <config_map_name>
  ```

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: <pod_name>
  spec:
    containers:
    - name: <container_name>
      image: <image>
      env:
      - name: <key>
        value: <value>
      ...
  ```

### Secret

- Stores sensitive information like API keys or passwords
- Secret definition

  ```yaml
  apiVersion: v1
  kind: Secret
  metadata:
    name: <secret_name>
  data:
    <key_hashed_value_pairs>
  ```
  
  Create with `kubectl create -f <yaml_file>`
  - Encoding value in linux

    ```bash
    echo -n <value> | base64
    ```

  - Decoding hashed value in linux
  
    ```bash
    echo -n <hashed_value> | base64 --decode
    ```

- Creating secret

  ```bash
  kubectl create secret generic <secret_name> --from-literal=<key>=<value>
  ```
  
  ```bash
  kubectl create secret generic <secret_name> --from-file=<secret_file>
  ```
  
- Injecting secret into pod

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: <pod_name>
  spec:
    containers:
    - name: <container_name>
      image: <image_name>
      envFrom:
      - secretRef:
          name: <config_map_name>
  ```

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: <pod_name>
  spec:
    containers:
    - name: <container_name>
      image: <image_name>
      volumes:
      - name: <volume_name>
        secret:
          secretName: <secret_name>
      ...
  ```

### Security Context

- Docker security
  - Container has its own namespace and can see its processes only
  - Processes run with root user by default
  - Root user within a container has limited privileges (different from the root in host)
    - Providing additional privileges

      ```bash
      docker run --cap-add <capability_name> <image_name>
      ```

    - Dropping privileges

      ```bash
      docker run --cap-drop <capability_name> <image_name>
      ```

    - Running container with all privileges enables

      ```bash
      docker run --privileged <image_name>
      ```

- Kubernetes security
  - Pod security context definition

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: <pod_name>
    spec:
      securityContext:
        runAsUser: [user_name]
      containers:
      - name: <container_name>
        image: <image>
    ```

  - Container security context definition

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: <pod_name>
    spec:
      containers:
      - name: <container_name>
        image: <image_name>
        securityContext:
          runAsUser: [user_name]
          capabilities:
            add: [capability_list]  # ["MAC_ADMIN"]
    ```

### Service Account

- Accounts used by applications to interact with Kubernetes clusters
- Creating service account
  
  ```bash
  kubectl create serviceaccount <service_account_name>
  ```

  - Also automatically creates tokens and secrets to store them
  - Can export tokens manually, or store in volumes for internal use
- Injecting service account into pod

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: <pod_name>
  spec:
    containers:
    - name: <container_name>
      image: <image_name>
    serviceAccount: <service_account_name>
    automountServiceAccountToken: false   # To disable default service account mount
  ```

### Resource Requirement

- Each container has resource request (CPU, memory, disk)
- Each container cannot use more than resource limit
  - Throttles CPU if exceeds limit
  - Allows memory to exceed limit, yet pod terminates if it cannot handle anymore
- Injecting resource requirement into pod

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: <pod_name>
  spec:
    containers:
    - name: <container_name>
      image: <image_name>
      resources:
        requests:
          memory: [memory_amount]  # "256M"
          cpu: [cpu_count]  # 1
        limits:
          memory: [max_memory_amount]  # "2G"
          cpu: [max_cpu_count]  # 5
  ```

### Taint and Toleration

- In order to run pods in certain nodes for specific purposes
- By default, master node is tainted so that pods cannot be run
- Tainting node
  
  ```bash
  kubectl taint node <node_name> <key>=<value>:<NoSchedule | PreferNoSchedule | NoExecute>
  ```

  - Taint effect is what happens to pods that do not tolerate the taint
    - `NoSchedule` - pods will not be scheduled on the node
    - `PreferNoSchedule` - system will try to avoid placing pod on the node but not guaranteed
    - `NoExecute` - only new pods will not be scheduled on the node
- Removing taint

  ```bash
  kubectl taint node <node_name> <taint_name>-
  ```

- Viewing node taints

  ```bash
  kubectl describe node <node_name> | grep Taints
  ```

- Adding toleration to pod

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: <pod_name>
  spec:
    containers:
    - name: <container_name>
      image: <image_name>
    tolerations:
    - key: <key>
      operator: <operator>
      value: <value>
      effect: <taint_effect>
  ```

### Node Selector

- In order to set a limitation on pods so that they only run on particular nodes
- Node selector definition
  
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: <pod_name>
  spec:
    containers:
    - name: <container_name>
      image: <image_name>
    nodeSelector:
      <key_value_pairs>
  ```

- Labeling node
  
  ```bash
  kubectl label node <node_name> <key>=<value>
  ```

### Node Affinity

- Provides advanced expressions for selecting particular nodes
- Node affinity definition

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: <pod_name>
  spec:
    containers:
    - name: <container_name>
      image: <image_name>
    affinity:
      nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
            - key: <key>
              operator: <In | NotIn | Exists | DoesNotExist | Gt | Lt>
              values: [value_list]
        preferredDuringSchedulingIgnoredDuringExecution:
          ...
        requiredDuringSchedulingRequiredDuringExecution:
          ...
        preferredDuringSchedulingRequiredDuringExecution:
          ...
  ```

  - `DuringScheduling` - state where pod does not exist and is created for the first time
    - If required, and the affinity rules are not matched, then the pod will not be scheduled
  - `DuringExecution` - state where pod has been running, and change is made in the environment
    - If required, and the affinity rules are not matched, then the running pod will be evicted or terminated

## Multi-Container Pod

### Design Patterns

- Sidecar
  - Deploying a logging agent alongside a web server to collect logs and forward them to the central log server
- Adaptor
  - Before sending logs to the central server, converts them to a common format
- Ambassador
  - Modifies connectivity in application code depending on the environment (development, test, production)

## Observability

- Pod status
  - `Pending` - when scheduler tries to figure out where to place the pod
  - `ContainerCreating` - when images are pulled to create container
  - `Running` - when container starts
- Pod conditions
  - `PodScheduled` - true when the pod is scheduled
  - `Initialized` - true when the pod is initialized
  - `ContainersReady` - true when containers in the pod are ready
  - `Ready` - true when the pod is ready
  
### Readiness Probe

- Checks whether application inside a container is actually ready
- Readiness probe definition
  - HTTP Test

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: <pod_name>
    spec:
      containers:
      - name: <container_name>
        image: <image_name>
        ports:
        - containerPort: <container_port>
        readinessProbe:
          httpGet:
            path: <path_to_readiness_check>
            port: <container_port>
          initialDelaySeconds: [delay_before_readiness_check]
          periodSeconds: [how_often_to_perform_readiness_check]
          failureThreashold: [num_attempts_before_stopping_probe]
    ```

  - TCP Test

    ```yaml
    ...
    readinessProbe:
      tcpSocket:
        port: <tcp_port>
    ```
  
  - Executing command

    ```yaml
    ...
    readinessProbe:
      exec:
        command:
          [command_list]
    ```

### Liveness Probe

- Checks whether application inside a container is actually healthy
- Liveness probe definition

  - HTTP Test

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: <pod_name>
    spec:
      containers:
      - name: <container_name>
        image: <image_name>
        ports:
        - containerPort: <container_port>
        livenessProbe:
          httpGet:
            path: <path_to_readiness_check>
            port: <container_port>
          initialDelaySeconds: [delay_before_liveness_check]
          periodSeconds: [how_often_to_perform_liveness_check]
          failureThreashold: [num_attempts_before_stopping_probe]
    ```

  - TCP Test

    ```yaml
    ...
    livenessProbe:
      tcpSocket:
        port: <tcp_port>
    ```
  
  - Executing command

    ```yaml
    ...
    livenessProbe:
      exec:
        command:
          [command_list]
    ```

### Container Logging

- Viewing live logs

  ```bash
  kubectl logs -f <pod_name> [container_name]
  ```

### Monitoring - Metrics Server

- In-memory monitoring solution
- Deploying metrics server

  ```bash
  git clone https://github.com/kubernetes-incubator/metrics-server.git
  kubectl create -f <yaml_repository>
  ```

- Viewing node performance metrics

  ```bash
  kubectl top nodes
  ```

- Viewing pod performance metrics

  ```bash
  kubectl top pods
  ```

## Pod Design

### Label, Selector, and Annotation

- Label
  - Properties attached to each object
  - Label definition

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: <pod_name>
      labels:
        <key_value_pairs>
    spec:
      containers:
      - name: <container_name>
        image: <image_name>
    ```

- Selector
  - Filters objects based on labels
  - Selecting specific pods

    ```bash
    kubectl get pods --selector <key>=<value>
    ```

- Annotation
  - Records details for informetric purpose
  - Annotation definition

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: <pod_name>
      annotations:
        <key_value_pairs>
    spec:
      containers:
      - name: <container_name>
        image: <image_name>
    ```

### Rolling Update and Rollback

- Rollout and Versioning
  - Helps keep track of changes made to deployment
  - Viewing rollout status

    ```bash
    kubectl rollout status deployment <deployment_name>
    ```

  - Viewing rollout history

    ```bash
    kubectl rollout history deployment <deployment_name> --revision=[revision_name]
    ```

  - Rollout with details

    ```bash
    kubectl rollout deployment <deployment_name> --record=true
    ```

- Deployment strategy
  - Recreate
    - Applies changes to all pods altogether
    - Application goes down and becomes inaccessible to users
  - Rolling update
    - Applies changes to pods one by one
    - Default strategy of Kubernetes deployment
- Updating deployment
  
  ```bash
  kubectl apply -f <yaml_file>
  ```

  ```bash
  kubectl set image <deployment_name> <old_image_name>=<new_image_name>
  ```

- Rollback

  ```bash
  kubectl rollout undo deployment <deployment_name>
  ```
  
### Job

- Runs a set of pods to perform a task to completion
- By default, pod always recreates its containers when they exit
  
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: <pod_name>
  spec:
    containers:
    - name: <container_name>
      image: <image_name>
    restartPolicy: Always | Never | OnFailure
  ```
  
- Job definition

  ```yaml
  apiVersion: batch/v1
  kind: job
  metadata:
    name: <job_name>
  spec:
    template:
      <pod_definition>
    completions: [num_pods_to_complete]
    parallelism: [num_pods_to_run_in_parallel]
  ```
  
  Create with `kubectl create -f <yaml_file>`
  - Job tries to create new pods until it has the given number of successful completions
- CronJob definition

  ```yaml
  apiVersion: batch/v1beta1
  kind: CronJob
  metadata:
    name: <job_name>
  spec:
    schedule: <job_schedule>
    jobTemplate:
      spec:
        template:
          <pod_definition>
        completions: [num_pods_to_complete]
        parallelism: [num_pods_to_run_in_parallel]
  ```

## Service and Networking

### Service

- NodePort

  <p align="center">
    <img src="https://github.com/kyminbb/ckad-prep/blob/main/docs/images/nodeport.png" width="70%" height="70%">
  </p>

  - Enables applications to be accesible to users
  - Listens to a port on a node and forwards requests on that port to inner pods
  - Automatically spans across multiple nodes and forwards requests randomly
  - NodePort definition
  
    ```yaml
    apiVersion: v1
    kind: Serivce
    metadata:
      name: <service_name>
    spec:
      type: NodePort
      ports:
      - targetPort: [pod_port]
        port: <service_port>
        nodePort: [node_port]  # 30000 - 32767
      selector:
        [key_value_pairs_of_pods_to_manage]
    ```

    Create with `kubectl create -f <yaml_file>`
- ClusterIP
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
      - targetPort: [pod_port]
        port: <service_port>
      selector:
        [key_value_pairs_of_pods_to_manage]
    ```

### Ingress Networking

<p align="center">
  <img src="https://github.com/kyminbb/ckad-prep/blob/main/docs/images/ingress.png" width="70%" height="70%">
</p>

- Helps users access applications using a single externally accessible URL
- Routes requests to different services based on URL part
- Implements SSL security
- Ingress controller
  - ex) Nginx, GCE
  - Config map definition

    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: nginx-configuration
    data:
      <key_value_pairs>
    ```

  - Deployment definition

    ```yaml
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      name: nginx-ingress-controller
    spec:
      replicas: 1
      selector:
        matchLabels:
          name: nginx-ingress
      template:
        metadata:
          labels:
            name: nginx-ingress
        spec:
          containers:
          - name: nginx-ingress-controller
            image: <nginx_ingress_controller_image_name>
          args:
          - /nginx-ingress-controller
          - --configmap=$(POD_NAMESPACE)/nginx-configuration
          env:
          - name: POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: POD_NAMESPACE
            valueFrom:
              fieldRef:
                fieldPath: metadata.namespace
          ports:
          - name: http
            containerPort: 80
          - name: https
            containerPort: 443
    ```

  - NodePort definition
  
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: nginx-ingress
    spec:
      type: NodePort
      ports:
      - port: 80
        targetPort: 80
        protocol: TCP
        name: http
      - port: 443
        targetPort: 443
        protocol: TCP
        name: https
      selector:
        name: nginx-ingress
    ```

  - Requires service account with correct cluster roles and role bindings
- Ingress resource
  
  <p align="center">
    <img src="https://github.com/kyminbb/ckad-prep/blob/main/docs/images/ingress-resource.png" width="70%" height="70%">
  </p>
  
  - Configures rules to foward requests
  - Ingress resource definition

    ```yaml
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: <ingress_resource_name>
    spec:
      rules:
      - host: <url_host>
        http:
          paths:
          - path: <url_path>
            backend:
              serviceName: <service_name>
              servicePort: <service_port>
          ...
      ...
    ```
  
    Create with `kubectl create -f <yaml_file>`

### Network Policy

<p align="center">
  <img src="https://github.com/kyminbb/ckad-prep/blob/main/docs/images/network-policy.png" width="60%" height="60%">
</p>

- Restricts communication between pods
  - By default, all pods in cluster can communicate with each other
- Network policy definition

  ```yaml
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: <network_policy_name>
  spec:
    podSelector:
      matchLabels:
        [key_value_pairs_of_pods_to_manage]
    policyTypes:
    - Ingress
    - Egress
    ingress:
    - from:  # (podSelector and namespaceSelector) or ipBlock
      - podSelector:
          matchLabels:
            [key_value_pairs_of_from_pods]
        namespaceSelector:  # Put dash to make (podSelector or namespaceSelector)
          matchLabels:
            [key_value_pairs_of_from_namespaces]
      - ipBlock:
          cidr: [allowed_ip_block]
          except: [unallowed_ip_block]
      ports:
        protocol: [ingress_protocol]
        port: [ingress_port]
    egress:
    - to:
      ...
  ```

  Create with `kubectl create -f <yaml_file>`
