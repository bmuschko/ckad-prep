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

## Adapter pattern

## Ambassador pattern
