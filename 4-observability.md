# Observability (18%)

## Defining a Pod’s Readiness and Liveness Probe

1. Create a new Pod named `hello` with the image `bonomat/nodejs-hello-world` that exposes the port 3000. Provide the name `nodejs-port` for the container port.
2. Add a Readiness Probe that checks the URL path / on the port referenced with the name `nodejs-port` after a 2 seconds delay. You do not have to define the period interval.
3. Add a Liveness Probe that verifies that the app is up and running every 8 seconds by checking the URL path / on the port referenced with the name `nodejs-port`. The probe should start with a 5 seconds delay.
4. Shell into container and curl `localhost:3000`. Write down the output. Exit the container.
5. Retrieve the logs from the container. Write down the output.

<details><summary>Show Solution</summary>
<p>

Create the intial YAML with the following command.

```bash
$ kubectl run hello --image=bonomat/nodejs-hello-world --restart=Never --port=3000 -o yaml --dry-run > pod.yaml
```

Edit the YAML file and add the probes.

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: hello
  name: hello
spec:
  containers:
  - image: bonomat/nodejs-hello-world
    name: hello
    ports:
    - name: nodejs-port
      containerPort: 3000
    readinessProbe:
      httpGet:
        path: /
        port: nodejs-port
      initialDelaySeconds: 2
    livenessProbe:
      httpGet:
        path: /
        port: nodejs-port
      initialDelaySeconds: 5
      periodSeconds: 8
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

Create the Pod from the YAML file, shell into the Pod as soon as it is running and execute the `curl` command.

```bash
$ kubectl create -f pod.yaml
pod/hello created
$ kubectl exec hello -it -- /bin/sh
/ # curl localhost:3000
<!DOCTYPE html>
<html>
<head>
	<title>NodeJS Docker Hello World</title>
	<meta charset="utf-8">
	<meta name="viewport" content="width=device-width, initial-scale=1">
	<link href="http://cdn.bootcss.com/bootstrap/3.3.2/css/bootstrap.min.css" rel="stylesheet">
	<link href="/stylesheets/styles.css" rel="stylesheet">
</head>
<body>
	<div class="container">
		<div class="well well-sm">
			<h2>This is just a hello world message</h2>
			<img a href="./cage.jpg"/>
			<img src="src/cage.jpg" alt="Smiley face" width="640">
		</div>
	</div>
</body>
</html>
/ # exit

$ kubectl logs pod/hello
Magic happens on port 3000
```

</p>
</details>

## Fixing a Misconfigured Pod

1. Create a new Pod with the following YAML.

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: failing-pod
  name: failing-pod
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - while true; do echo $(date) >> ~/tmp/curr-date.txt; sleep
      5; done;
    image: busybox
    name: failing-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

2. Check the Pod's status. Do you see any issue?
3. Follow the logs of the running container and identify an issue.
4. Fix the issue by shelling into the container. After resolving the issue the current date should be written to a file. Render the output.

<details><summary>Show Solution</summary>
<p>

First, create the Pod with the given YAML content.

```bash
$ vim pod.yaml
$ kubectl create -f pod.yaml
```

The Pod seems to be running without problems.

```bash
$ kubectl get pods
NAME          READY   STATUS    RESTARTS   AGE
failing-pod   1/1     Running   0          5s
```

Render the logs of the container. The output should indicate an error message every 5 seconds.

```bash
$ kubectl logs failing-pod
Unable to write file!
/bin/sh: 1: cannot create /root/tmp/curr-date.txt: Directory nonexistent
Unable to write file!
/bin/sh: 1: cannot create /root/tmp/curr-date.txt: Directory nonexistent
Unable to write file!
/bin/sh: 1: cannot create /root/tmp/curr-date.txt: Directory nonexistent
```

Apparently, the directory we want to write to does not exist. Log into the container and create the directory. The file `~/tmp/curr-date.txt` is populated.

```bash
$ kubectl exec failing-pod -it -- /bin/sh
/ # mkdir -p ~/tmp
/ # cd ~/tmp
/ # ls -l
total 4
-rw-r--r-- 1 root root 112 May  9 23:52 curr-date.txt
/ # cat ~/tmp/curr-date.txt
Thu May 9 23:59:01 UTC 2019
Thu May 9 23:59:06 UTC 2019
Thu May 9 23:59:11 UTC 2019
/ # exit
```

</p>
</details>