# Services & Networking (13%)

## Exposing a service

1. Create a deployment named `myapp` that creates 2 replicas for Pods with the image `nginx`. Expose the container port 80.
2. Create a temporary Pods using the image `busybox` and run a `wget` command against the other Pod with port 80.
3. Expose the deployment so that requests can be made against the service from outside of the cluster.
4. Run a `curl` command against the service instead of the Pods directly.
5. (Optional) Can you expose the Pods as a service without a deployment?

<details><summary>Show Solution</summary>
<p>

Create a deployment with 2 replicas first. You should end up with one deployment and two Pods.

```bash
$ kubectl run myapp --image=nginx --restart=Always --replicas=2 --port=80
deployment.apps/myapp created
$ kubectl get deployments,pods
NAME                          DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/myapp   2         2         2            2           59s

NAME                         READY   STATUS    RESTARTS   AGE
pod/myapp-7bc568bfdd-972wg   1/1     Running   0          59s
pod/myapp-7bc568bfdd-l5nmz   1/1     Running   0          59s
```

You can retrieve the IP addresses for each Pod with the `-o wide` option.

```bash
$ kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE   IP               NODE
myapp-7bc568bfdd-972wg   1/1     Running   0          2m    192.168.60.136   docker-for-desktop
myapp-7bc568bfdd-l5nmz   1/1     Running   0          2m    192.168.60.135   docker-for-desktop
$ kubectl run tmp --image=busybox --restart=Never -it --rm -- wget -O- 192.168.60.136:80
Connecting to 192.168.60.136:80 (192.168.60.136:80)
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
-                    100% |********************************|   612  0:00:00 ETA
pod "tmp" deleted
```

Expose the service with the type `NodePort` and the target port 80. Determine the cluster IP (or localhost if you use Kubernetes for Docker Desktop) and use it for the `curl` command.

``` bash
$ kubectl expose deploy myapp --type=NodePort --target-port=80
service/myapp exposed
$ kubectl get services
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
myapp        NodePort    10.106.149.12   <none>        80:31837/TCP   49s
$ curl localhost:31837
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

</p>
</details>

## Defining a network policy

Let's assume we are working on an application stack that defines three different layers: a frontend, a backend and a database. Each of the layers runs in a Pod. You can find the definition in the YAML file `app-stack.yaml`. The application needs to run in the namespace `app-stack`.

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: frontend
  namespace: app-stack
  labels:
    app: todo
    tier: frontend
spec:
  containers:
    - name: frontend
      image: nginx

---

kind: Pod
apiVersion: v1
metadata:
  name: backend
  namespace: app-stack
  labels:
    app: todo
    tier: backend
spec:
  containers:
    - name: backend
      image: nginx

---

kind: Pod
apiVersion: v1
metadata:
  name: database
  namespace: app-stack
  labels:
    app: todo
    tier: database
spec:
  containers:
    - name: database
      image: mysql
      env:
      - name: MYSQL_ROOT_PASSWORD
        value: example
```

1. Create the required namespace.
2. Create a network policy in the YAML file `app-stack-network-policy.yaml`.
3. The network policy should allow incoming traffic from the backend to the database but disallow incoming traffic from the frontend.
4. Incoming traffic to the database should only be allowed on TCP port 3306 and no other port.

<details><summary>Show Solution</summary>
<p>

```
$ kubectl create namespace app-stack
namespace/app-stack created
$ kubectl create -f app-stack.yaml
pod/frontend created
pod/backend created
pod/database created
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-stack-network-policy
  namespace: app-stack
spec:
  podSelector:
    matchLabels:
      app: todo
      tier: database
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: todo
          tier: backend
    ports:
    - protocol: TCP
      port: 3306
```

</p>
</details>