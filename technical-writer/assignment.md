# Debug Commands in Kubernetes
## General Information
For debugging, use the Kubernetes Command Line Interface, kubectl, which interacts with a cluster's control plane through the Kubernetes API. Use the following syntax to execute kubectl commands from your terminal window:
```
kubectl [command] [TYPE] [NAME] [flags]
```
For more details on each parameter, see the [Command line tool (kubectl)](https://kubernetes.io/docs/reference/kubectl/) reference.
For a complete list of available flags for each command, see the [Kubectl Reference Docs](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-strong-getting-started-strong-) reference.

We recommend using the following commands for debugging:
- `kubectl get pods` to identify potential issues.
- `kubectl describe pod` and `kubectl get events` to gather diagnostic details.
- `kubectl logs` to analyze application errors.
- `kubectl exec` for interactive debugging.
- `kubectl debug` for deep investigation.

See the descriptions of commands below.
## kubectl get pods
Start debugging with `kubectl get pods` to list all pods and their statuses.

The command syntax:
```
kubectl get pods [NAME] [flags]
```
|Parameter|Required?|Description|
|---|---|---|
|[TYPE], value = `pods`|Yes|Specifies that you are retrieving information about pods.|
|[NAME]|No|If provided, the command retrieves details for a specific pod.|
|[flags]|No|Additional options.<br>For example, you can use `--namespace` to specify the namespace. If this flag is not provided, the command defaults to the current namespace set in your context.|

Example command to list all pods in `my-namespace`:
```
kubectl get pods --namespace my-namespace
```

When analyzing the command output, we recommend paying attention to the following cases:
- `STATUS` parameter value is `CrashLoopBackOff`, `Error`, `OOMKilled`, `ImagePullBackOff`, `Evicted`, `RunContainerError`, `CreateContainerConfigError`, `Init:CrashLoopBackOff`, `Init:Error`, `NodeAffinity`, `Terminating`, or `Unknown`.
- `STATUS` parameter value is `Pending` or `ContainerCreating` for too long.
- `READY` parameter value is `0/X` for too long.
- `RESTARTS` parameter value keeps growing.
- `AGE` parameter value keeps resetting.

Example output: 
```
NAME                READY   STATUS             RESTARTS   AGE
web-app-123         1/1     Running            0          3m
api-server-789      1/1     Running            2          10m
database-service    0/1     CrashLoopBackOff   5          15m
```
## kubectl describe pod
Use `kubectl describe pod` to view detailed pod information, including events, conditions, node placement, scheduling or resource issues, container restarts, and failures.

The command syntax:
```
kubectl describe pod [NAME] [flags]
```
|Parameter|Required?|Description|
|---|---|---|
|[TYPE], value = `pod`|Yes|Specifies that you are retrieving information about a pod.|
|[NAME]|Yes|Name of the pod you want to get the information about.|
|[flags]|No|Additional options, for example, `--namespace`.<br>You can also use `--show-events` to see the events section, even if your cluster version or configuration does not show it by default.|

Example command to retrieve information for `my-pod` in `my-namespace`, including the events section:
```
kubectl describe pod my-pod --namespace my-namespace --show-events
```
We recommend analyzing the command output for scheduling issues (such as `FailedScheduling`), container failures (such as `CrashLoopBackOff`), and image pull failures (such as `ErrImagePull`), as well as resource allocation and network issues.

## kubectl get events
Use the `kubectl get events` command to view a timeline of cluster-wide or namespace-specific events, including scheduling or resource issues, failed container restarts, and other system-related warnings or errors.

The command syntax:
```
kubectl get events [flags]
```
|Parameter|Required?|Description|
|---|---|---|
|[TYPE], value = `events`|Yes|Specifies that you are retrieving information about events.|
|[flags]|No|Additional options, for example, `--namespace`.<br>You can also use `--since` to filter events that occurred within a specific time range.|

Example command to retrieve events in `my-namespace` from the last 10 minutes:
```
kubectl get events --namespace my-namespace --since=10m
```
When analyzing the command output, we recommend paying attention to events with `TYPE` parameter value `Warning`.
## kubectl logs
Use the kubectl logs command to view application-level errors, such as missing config, bad dependencies, or failed connections.

The command syntax:
```
kubectl logs [NAME] [flags]
```
|Parameter|Required?|Description|
|---|---|---|
|[NAME]|Yes|Name of the pod that you want to retrieve the logs for.|
|[flags]|Yes, if the pod has multiple containers, `--container` flag is required.<br>No, if the pod has one container.|Additional options, for example, `--namespace`.<br>You can also use `--since-time` to fetch logs from an exact moment in time.|

Example command to retrieve logs for `my-pod` from February 25, 2025, 14:09 UTC:
```
kubectl logs my-pod --since-time=2025-02-25T14:09:00Z
```
We recommend analyzing the command output for error messages, abnormal exit codes (such as 137 - `OOMKilled`), frequent container restarts, failed health checks, resource exhaustion, connection or network issues, service unavailability, warnings, and performance bottlenecks.
## kubectl exec
Use `kubectl exec` to execute commands within containers and inspect environment variables, network connections, and file system issues. This command is especially useful when logs do not provide sufficient information, for example, when logs are stored in files and are not available in the container's standard output.

The command syntax:
```
kubectl exec [NAME] [flags] -- COMMAND [args...]
```
|Parameter|Required?|Description|
|---|---|---|
|[NAME]|Yes|Name of the pod that you want to execute the command in.|
|[flags]|Yes, if the pod has multiple containers, `--container` flag is required.<br>No, if the pod has one container.|Additional options.|
|--|Yes|Separates kubectl options from the command.|
|COMMAND|Yes|The actual command to execute inside the container.|
|[args...]|Depends on the COMMAND.|COMMAND arguments.|

Example command to execute the `cat` COMMAND (to print the contents of a file) with `/var/log/my-pod/system.log` argument (file path) in `my-pod`:
```
kubectl exec my-pod -- cat /var/log/my-pod/system.log
```
The output and analysis process depend on the executed COMMAND.
## kubectl debug
Use `kubectl debug` to create a temporary debug container with full tooling, especially when `kubectl exec` isn't sufficient - if the container is crashing, unresponsive, or lacks the required debugging tools.

The command syntax:
```
kubectl debug [NAME] [flags]
```
|Parameter|Required?|Description|
|---|---|---|
|[NAME]|Yes|Name of the pod that you want to debug.|
|[flags]|Yes.<br>`--image` flag is required to specify the image used for the debug container.<br>If the pod has multiple containers, `--target` flag is required to specify the target container.|Additional options.<br>For example, you can use `-it` to enable interactive terminal mode.|

Example command to create an interactive debugging session in `my-pod` using `my-image`:
```
kubectl debug my-pod -it --image=my-image
```
To explore possible kubectl debug use cases, see the [Debug Running Pods](https://kubernetes.io/docs/tasks/debug/debug-application/debug-running-pod/) and [Kubectl Reference Docs](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-strong-getting-started-strong-) references.
## References

- [Command line tool (kubectl)](https://kubernetes.io/docs/reference/kubectl/)
- [Debug Running Pods](https://kubernetes.io/docs/tasks/debug/debug-application/debug-running-pod/)
- [Kubectl Reference Docs](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-strong-getting-started-strong-)
