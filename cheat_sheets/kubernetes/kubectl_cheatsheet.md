---
tags: kuberentes, kubectl, cheatsheet
---
# kubectl cheat sheet

Personal reference guide that provides a memo of commonly used commands and
shortcuts for the Kubernetes command-line tool, `kubectl`.

---
## Force pod deletion

You can use following command to delete a pod forcefully:

```bash
kubectl delete pod <pod_name> --grace-period=0 --force --namespace <namespace>
```

## Scale deployments manually

Scaling a Kubernetes deployment involves adjusting the number of replicas or
pods running your application:

```sh
kubectl scale deployment <deployment-name> --replicas=<desired-replica-count>
```

## Restart a deployment

This method gradually replaces the old pods with new ones, ensuring that your application
remains available during the process

```sh
kubectl rollout restart deployment <deployment-name>
```

## Troubleshooting with a busybox container

Occasionally, it proves beneficial to initiate a temporary shell for
troubleshooting purposes. You can achieve this by:

```sh
kubectl run tmp-shell --rm -it --image busybox -n <namespace> -- /bin/sh
```

**_Note_**: to connect to containers using the Kubernetes internal DNS system: 
`<service-name>.<namespace>.svc.cluster.local`

## Manually starting a job from a CronJobs

To manually run a CronJob as a Job you run the `kubectl create job` command.
You then specify the CronJob to base the job off of using the `--from` flag.
Lastly, you specify a unique name for the job.

```sh
kubectl create job --from=cronjob/postgres-backup postgres-backup-manual-01
```

## List all containers running in a pod

The following command ouputs a list of containers running in a specific pod:

```sh
kubectl get pods <pod> -o jsonpath='{.spec.containers[*].name}' -n <namespace>
```

## Read a Kubernetes secret and decoded in one line

To read a Kubernetes secret and decode it in one line, you can use kubectl along
with the base64 utility. Here's an example command that reads a specific key
from a secret, decodes it, and outputs the value:

```sh
kubectl get secret <SECRET_NAME> -o jsonpath='{.data.<KEY>}' | base64 -d
```
Replace `<SECRET_NAME>` with the name of your Kubernetes secret and `<KEY>`
with the key you want to read from the secret.

## List the labels of a pod

The command to get the labels of a pod is:

```bash
kubectl get pod <pod> -n <namespace> -o=jsonpath='{.metadata.labels}'
```

## Delete a pod by a label:

```bash
kubectl delete pod -l <label-key>=<label-valu> -n <namespace>
```

## List all pods running in a worker node

```bash

kubectl get pods --all-namespaces --field-selector spec.nodeName=<node>

```

## Cordon a node

To "cordon" a Kubernetes node means to mark the node as unschedulable. This
action prevents new pods from being scheduled onto that node but does not affect
the pods that are already running on the node:

The command to cordon a node in Kubernetes is:

```bash
kubectl cordon <node>
```

To reverse the process and allow scheduling on the node again:

```bash
kubectl uncordon <node>
```

## Drain a node

Draining a node in Kubernetes involves safely evicting all pods from the
specified node, with the intention of preparing the node for maintenance or
decommissioning. When you drain a node, Kubernetes respects the
PodDisruptionBudgets (if set), and it attempts to gracefully terminate the pods,
giving them time to shut down properly. Here's how you can drain a node:

1) **Cordon** the node (optional but recommended to prevent new pods from being
scheduled on it while you're draining it):

```bash
kubectl cordon <node>
```

2) **Drain** the node:
```
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data
```

`--ignore-daemonsets:` This flag tells `kubectl` to ignore DaemonSet-managed
pods. DaemonSets are designed to provide a pod per node for system services and
are automatically rescheduled by Kubernetes, so they are not evicted by the
drain command.

`--delete-emptydir-data`: This flag is necessary if any pods on the node use
emptyDir volumes. It acknowledges that you are okay with deleting the data in
emptyDir volumes. Be cautious with this flag, as it results in data loss for
those volumes.

## Delete Jobs that are marked as completed

To delete Jobs that are marked as completed:
```bash
kubectl delete jobs --field-selector=status.successful=1
```
## List of some common resource types and their short names

Short Name | Long Name
---|---
`configmaps`|`cm`
`customresourcedefinitions`|`crd`
`daemonsets`|`ds`
`deployments`|`deploy`
`endpoints`|`ep`
`horizontalpodautoscalers`|`hpa`
`ingresses`|`ing`
`jobs`|`job`
`namespaces`|`ns`
`networkpolicies`|`netpol`
`nodes`|`no`
`persistentvolumeclaims`|`pvc`
`persistentvolumes`|`pv`
`pods`|`po`
`replicasets`|`rs`
`replicationcontrollers`|`rc`
`roles`|`role`
`rolebindings`|`rolebinding`
`secrets`|`secret`
`services`|`svc`
`statefulsets`|`sts`
`storageclasses`|`sc`
