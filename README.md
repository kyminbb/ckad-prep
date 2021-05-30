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
            memory: [memory_amount] # "256M"
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
  kubectl taint node <node_name> <key>=<value>:<taint_effect>
  ```

  - Taint effect is what happens to pods that do not tolerate the taint 
    - NoSchedule - pods will not be scheduled on the node
    - PreferNoSchedule - system will try to avoid placing pod on the node but not guaranteed
    - NoExecute - only new pods will not be scheduled on the node
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
              operator: <operator>
              values: [value_list]
        preferredDuringSchedulingIgnoredDuringExecution:
          ...
        requiredDuringSchedulingRequiredDuringExecution:
          ...
        preferredDuringSchedulingRequiredDuringExecution:
          ...
  ```

  - DuringScheduling - state where pod does not exist and is created for the first time
    - If required, and the affinity rules are not matched, then the pod will not be scheduled
  - DuringExecution - state where pod has been running, and change is made in the environment
    - If required, and the affinity rules are not matched, then the running pod will be evicted or terminated
