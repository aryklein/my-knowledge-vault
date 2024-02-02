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

The following command terminates the existing pods and creates new ones in
their place:

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
