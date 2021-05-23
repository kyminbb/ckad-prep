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
