# Multi-Container Pods (10%)

## Pods with multiple containers

1. Declare a new Pod with the name `multi` that uses the image `nginx` in the file `pod.yaml`. Do not create the Pod yet.
2. Edit the YAML file and add a second container named `busybox` that uses the image `busybox`.
3. Create the Pod and verify it has been created. Write the output to the file named `multi-container-list.txt`.
4. Execute the command `ls` on the container `nginx` of the Pod. Write the output to the file named `multi-container-output.txt`. Exit the container.

<details><summary>Show Solution</summary>
<p>

```bash
$ kubectl run multi --image=nginx --restart=Never -o yaml --dry-run > pod.yaml
$ kubectl create -f pod.yaml
$ kubectl get pods
$ kubectl exec multi --container=nginx -it -- ls
```

</p>
</details>

## Sidecar pattern

Many applications use a configuration file for parameterizing the runtime behavior. For example, an application may need to connection to a database via URL and credentials or needs to set the default locale. Configure a multi-container Pod that implements the sidecar pattern.

1. Create a new Pod named `sidecar` in a YAML file named `sidecar.yaml`. The Pod declares two containers. The container `app` runs the application with the image `bmuschko/nodejs-hello-world:1.0.0`. The sidecar container `config` uses the image `nginx` and runs the command `while true; do echo 'Reading config file'; sleep 10; done;` to emulate a synchronization process.
2. Before creating the Pod, define an `emptyDir` volume. Mount the volume in both containers with the path `/var/app/config`.
3. Create the Pod, log into the container `config` and create the file named `app.yaml` in `/var/app/config`. Create the key/value pair `locale: en-US` and save contents of the file. Log out of the container.
4. Log into container `app` and navigate to the directory `/var/app/config`. Print out the contents of the file `/var/app/config/app.yaml`.

<details><summary>Show Solution</summary>
<p>

Start by generating the YAML file that defines the `app` container.

```bash
$ kubectl run sidecar --image=google/nodejs-hello --restart=Never -o yaml --dry-run > sidecar.yaml
```

Edit the file `sidecar.yml` and add the sidecar container with the appropriate command. Change the name of the `app` container. Furthermore, add the volume mount to both containers.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar
spec:
  containers:
  - image: bmuschko/nodejs-hello-world:1.0.0
    name: app
    volumeMounts:
      - name: config-volume
        mountPath: /var/app/config
  - image: nginx
    name: config
    args:
    - /bin/sh
    - -c
    - while true; do echo 'Reading config file'; sleep 10; done;
    volumeMounts:
      - name: config-volume
        mountPath: /var/app/config
    resources: {}
  volumes:
    - name: config-volume
      emptyDir: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

Create the Pod by evaluating the YAML file.

```bash
$ kubectl create -f sidecar.yaml
```

Log into the `config` container and create the `app.yaml` file.

```bash
$ kubectl exec sidecar --container=config -it -- /bin/sh
# cd /var/app/config
# echo 'locale: en-US' > app.yaml
# exit
```

Log into the `app` container and create the config file.

```bash
$ kubectl exec sidecar --container=app -it -- /bin/sh
\# cat /var/app/config/app.yaml
\# exit
```

</p>
</details>

## Adapter pattern

## Ambassador pattern
