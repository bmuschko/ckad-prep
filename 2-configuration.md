# Configuration (18%)

## Defining Pod resources

Create a resource quota named `apps` under the namespace `rq-demo` using the following YAML definition in the file `rq.yaml`.

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: app
spec:
  hard:
    pods: "2"
    requests.cpu: "2"
    requests.memory: 500m
```

Create a new Pod that exceeds the limits of the resource quota requirements. Write down the error message. Change the request limits to fulfill the requirements to ensure that the Pod could be created successfully. Write down the output of the command that renders the used amount of resources for the namespace.

<details><summary>Show Solution</summary>
<p>

First create the namespace and the resource quota in the namespace.

```
$ kubectl create namespace rq-demo
$ kubectl create -f rq.yaml --namespace=rq-demo
resourcequota/app created
$ kubectl describe namespace rq-demo --namespace=rq-demo
Name:         rq-demo
Labels:       <none>
Annotations:  <none>
Status:       Active

Resource Quotas
 Name:            app
 Resource         Used  Hard
 --------         ---   ---
 pods             0     2
 requests.cpu     0     2
 requests.memory  0     500m

No resource limits.
```

Next, create the YAML file named `pod.yaml` with more requested memory than available in the quota.

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: mypod
  name: mypod
spec:
  containers:
  - image: nginx
    name: mypod
    resources:
      requests:
        memory: "1G"
        cpu: "400m"
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

Create the Pod and observe the error message.

```
$ kubectl create -f pod.yaml --namespace=rq-demo
Error from server (Forbidden): error when creating "pod.yaml": pods "mypod" is forbidden: exceeded quota: app, requested: requests.memory=1G, used: requests.memory=0, limited: requests.memory=500m
```

Lower the memory settings to less than `500m` e.g. `200m`.

```
$ kubectl create -f pod.yaml --namespace=rq-demo
pod/mypod created
$ kubectl describe quota --namespace=rq-demo
Name:            app
Namespace:       rq-demo
Resource         Used  Hard
--------         ----  ----
pods             1     2
requests.cpu     400m  2
requests.memory  200m  500m
```

</p>
</details>

