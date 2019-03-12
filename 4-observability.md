# Observability (18%)

## Defining a readiness and liveness probe

1. Create a new Pod named `hello` with the image `bonomat/nodejs-hello-world` that exposes the port 3000. Provide the name `node-port` for the port.
2. Add a readiness probe that checks the URL path / on the port referenced with the name `node-port` after a 2 seconds delay. You do not have to define the period interval.
3. Add a liveness probe that verifies that the app is up and running every 8 seconds by checking the URL path / on the port referenced with the name `node-port`. The probe should start with a 5 seconds delay.
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
    - name: node-port
      containerPort: 3000
    readinessProbe:
      httpGet:
        path: /
        port: node-port
      initialDelaySeconds: 2
    livenessProbe:
      httpGet:
        path: /
        port: node-port
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
$ kubectl exec hello -it -- sh
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