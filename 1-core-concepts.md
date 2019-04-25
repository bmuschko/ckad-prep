# Core Concepts (13%)

## Creating a Pod and fixing issues

1. Create the namespace `ckad-prep`.
2. In the namespace `ckad-prep` create a new Pod named `mypod` with the image `nginx:2.3.5`. Expose the port 80.
3. Identify the issue with creating the container. Write down the root cause of issue in a file named `pod-error.txt`.
4. Change the image of the Pod to `nginx:1.15.12`.
5. List the Pod and ensure that the container is running.
6. Log into the container and run the `ls` command. Write down the output. Log out of the container.
7. Delete the Pod and namespace.

<details><summary>Show Solution</summary>
<p>

First, create the namespace.

```bash
$ kubectl create namespace ckad-prep
```

Next, create the Pod in the new namespace.

```bash
$ kubectl run mypod --image=nginx:2.3.5 --restart=Never --port=80 --namespace=ckad-prep
pod/mypod created
```

You will see that the image cannot be pulled as it doesn't exist with this tag.

```bash
$ kubectl get pod -n ckad-prep
NAME    READY   STATUS             RESTARTS   AGE
mypod   0/1     ImagePullBackOff   0          1m
```

The list of events can give you a deeper insight.

```bash
$ kubectl describe pod -n ckad-prep
...
Events:
  Type     Reason                 Age                 From                         Message
  ----     ------                 ----                ----                         -------
  Normal   Scheduled              3m3s                default-scheduler            Successfully assigned mypod to docker-for-desktop
  Normal   SuccessfulMountVolume  3m2s                kubelet, docker-for-desktop  MountVolume.SetUp succeeded for volume "default-token-jbcl6"
  Normal   Pulling                84s (x4 over 3m2s)  kubelet, docker-for-desktop  pulling image "nginx:2.3.5"
  Warning  Failed                 83s (x4 over 3m1s)  kubelet, docker-for-desktop  Failed to pull image "nginx:2.3.5": rpc error: code = Unknown desc = Error response from daemon: manifest for nginx:2.3.5 not found
  Warning  Failed                 83s (x4 over 3m1s)  kubelet, docker-for-desktop  Error: ErrImagePull
  Normal   BackOff                69s (x6 over 3m)    kubelet, docker-for-desktop  Back-off pulling image "nginx:2.3.5"
  Warning  Failed                 69s (x6 over 3m)    kubelet, docker-for-desktop  Error: ImagePullBackOff
```

Go ahead and edit the existing Pod. Alternatively, you could also just use the `set image` command.

```bash
$ kubectl edit pod mypod -n ckad-prep
```

After setting an image that does exist, the Pod should render the status `Running`.

```bash
$ kubectl get pod -n ckad-prep
NAME    READY   STATUS    RESTARTS   AGE
mypod   1/1     Running   0          14m
```

You can shell into the container and run the `ls` command.

```bash
$ kubectl exec mypod -it --namespace=ckad-prep  -- /bin/sh
/ # ls
bin  boot  dev	etc  home  lib	lib64  media  mnt  opt	proc  root  run  sbin  srv  sys  tmp  usr  var
/ # exit
```

Delete the Pod and namespace after you are done.

```bash
$ kubectl delete pod mypod --namespace=ckad-prep
pod "mypod" deleted
$ kubectl delete namespace ckad-prep
namespace "ckad-prep" deleted
```

</p>
</details>