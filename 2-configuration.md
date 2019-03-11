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

```bash
$ kubectl create namespace rq-demo
$ kubectl create -f rq.yaml --namespace=rq-demo
resourcequota/app created
$ kubectl describe quota --namespace=rq-demo
Name:            app
Namespace:       rq-demo
Resource         Used  Hard
--------         ----  ----
pods             0     2
requests.cpu     0     2
requests.memory  0     500m
```

Next, create the YAML file named `pod.yaml` with more requested memory than available in the quota.

```yaml
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

```bash
$ kubectl create -f pod.yaml --namespace=rq-demo
Error from server (Forbidden): error when creating "pod.yaml": pods "mypod" is forbidden: exceeded quota: app, requested: requests.memory=1G, used: requests.memory=0, limited: requests.memory=500m
```

Lower the memory settings to less than `500m` (e.g. `200m`) and create the Pod.

```bash
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

## Creating a security context

1. Create a Pod named `secured` that uses the image `nginx` for a single container. Mount an `emptyDir` volume to the directory `/data/app`.
2. Files created on the volume should use the group ID 3000.
3. Get a shell to the running container and create a new file named `logs.txt` in the directory `/data/app`. List the contents of the directory and write them down.

<details><summary>Show Solution</summary>
<p>

Start by creating the Pod definition as YAML file.

```bash
$ kubectl run secured --image=nginx --restart=Never -o yaml --dry-run > secured.yaml
```

Edit the YAML file, add a volume and a volume mount. Add a security context with the relevant group ID.

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secured
  name: secured
spec:
  securityContext:
    fsGroup: 3000
  containers:
  - image: nginx
    name: secured
    volumeMounts:
    - name: data-vol
      mountPath: /data/app
    resources: {}
  volumes:
  - name: data-vol
    emptyDir: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

Create the Pod and log into the container. Create the file in the directory of the volume mount. The group ID should be 3000 as defined by the security context.

```bash
$ kubectl create -f secured.yaml
pod/secured created
$ kubectl exec -it secured -- sh
/ # cd /data/app
/ # touch logs.txt
/ # ls -l
-rw-r--r-- 1 root 3000 0 Mar 11 15:56 logs.txt
/ # exit
```

</p>
</details>

## Using a service account

1. Create a new service account named `backend-team`.
2. Print out the token for the service account in YAML format.
3. Create a Pod named `backend` that uses the image `nginx` and the identity `backend-team` for running processes.

<details><summary>Show Solution</summary>
<p>

First, create the service acccount and inspect it.

```bash
$ kubectl create serviceaccount backend-team
serviceaccount/backend-team created
$ kubectl get serviceaccount backend-team -o yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: 2019-03-11T18:54:25Z
  name: backend-team
  namespace: default
  resourceVersion: "1114478"
  selfLink: /api/v1/namespaces/default/serviceaccounts/backend-team
  uid: 173435c1-442f-11e9-8dc3-025000000001
secrets:
- name: backend-team-token-dm2f
```

Next, you can create a new Pod and assign the service account to it.

```bash
$ kubectl run backend --image=nginx --restart=Never --serviceaccount=backend-team
```

</p>
</details>