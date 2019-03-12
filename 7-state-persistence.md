# State Persistence (8%)

## Using a volume from a Pod

1. Create a Persistent Volume named `pv`, access mode `ReadWriteMany`, storage class name `shared` and 512MB of storage capacity.
2. Create a Persistent Volume Claim named `pvc` that requests the Persistent Volume in step 1. The claim should request 256MB. Ensure that the Persistent Volume Claim is properly bound after its creation.
3. Mount the Persistent Volume Claim from a new Pod named `app` with the path `/var/app/config`. The Pod uses the image `nginx`.
4. Check the events of the Pod after starting it to ensure that the volume was mounted properly.

<details><summary>Show Solution</summary>
<p>

Create a YAML file for the Persistent Volume and create it with the command `kubectl create` command.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv
spec:
  capacity:
    storage: 512m
  accessModes:
    - ReadWriteMany
  storageClassName: shared
  hostPath:
    path: /data/config
```

Create a YAML file for the Persistent Volume Claim and create it with the command `kubectl create` command.

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 256m
  storageClassName: shared
```

Create a YAML file for the Pod and create it with the command `kubectl create` command.

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: app
  name: app
spec:
  containers:
  - image: nginx
    name: app
    volumeMounts:
      - mountPath: "/data/app/config"
        name: configpvc
    resources: {}
  volumes:
    - name: configpvc
      persistentVolumeClaim:
        claimName: pvc
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

You can check the events of a Pod with the `kubectl describe` command. You should see an entry that indicates the successful mount.

```bash
$ kubectl describe pod app
...
Events:
  Type    Reason                 Age   From                         Message
  ----    ------                 ----  ----                         -------
  Normal  Scheduled              16s   default-scheduler            Successfully assigned app to docker-for-desktop
  Normal  SuccessfulMountVolume  16s   kubelet, docker-for-desktop  MountVolume.SetUp succeeded for volume "pv"
  Normal  SuccessfulMountVolume  16s   kubelet, docker-for-desktop  MountVolume.SetUp succeeded for volume "default-token-fsmmp"
  Normal  Pulling                15s   kubelet, docker-for-desktop  pulling image "nginx"
  Normal  Pulled                 14s   kubelet, docker-for-desktop  Successfully pulled image "nginx"
  Normal  Created                14s   kubelet, docker-for-desktop  Created container
  Normal  Started                13s   kubelet, docker-for-desktop  Started container
```

</p>
</details>