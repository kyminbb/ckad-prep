# ckad-prep

## Configuration

### Commands and Arguments

- Overwrite commands and arguments of container that runs in pod

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
    labels:
      [key_value_pairs]
  spec:
    containers:
      - name: <container_name>
        image: <image>
        envFrom:
          - configMapRef:
              name: [config_map_name]
  ```

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
        env:
          - name: [key]
            value: [value]
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
    labels:
      [key_value_pairs]
  spec:
    containers:
      - name: <container_name>
        image: <image>
        envFrom:
          - secretRef:
              name: [config_map_name]
  ```

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
        volumes:
          - name: [volume_name]
            secret:
              secretName: [secret_name]
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
      labels: [key_value_pairs]
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
      labels: [key_value_pairs]
    spec:
      containers:
      - name: <container_name>
        image: <image>
        securityContext:
          runAsUser: [user_name]
          capabilities:
            add: [capability_list]  # ["MAC_ADMIN"]
    ```
