# Bonus Exercises

## Shortcuts and Time Savers

### Using an alias for kubectl

You might find it tedious to enter the `kubectl` every time you want to run a command. It helps a lot to set up an alias for the command.

```bash
$ alias k=kubectl
$ k version
Client Version: version.Info{Major:"1", Minor:"12", GitVersion:"v1.12.2", GitCommit:"17c77c7898218073f14c8d573582e8d2313dc740", GitTreeState:"clean", BuildDate:"2018-10-30T21:39:16Z", GoVersion:"go1.11.1", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.11", GitCommit:"637c7e288581ee40ab4ca210618a89a555b6e7e9", GitTreeState:"clean", BuildDate:"2018-11-26T14:25:46Z", GoVersion:"go1.9.3", Compiler:"gc", Platform:"linux/amd64"}
```

### Setting the namespace per context

At the beginning of each question you will be provided a command you need to run in order to perform the operations against a specific cluster. Run the command at the beginning of each question. Additionally, you can also [set the namespace for the question as a preference](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/#setting-the-namespace-preference) to avoid having to the type the `--namespace=<namespace-of-question>` parameter with every operation.

```bash
$ kubectl config set-context <context-of-question> --namespace=<namespace-of-question>
```

### Deleting Kubernetes objects quickly

You might make mistakes when creating objects and will want to delete and recreate it. Deleting objects sometimes takes a while as Kubernetes attempts to do it gracefully. You do not want to wait for a graceful shutdown during the exam due to the time constraint. It's much faster to force the deletion.

```bash
$ kubectl delete pod nginx --grace-period=0 --force
```

## Bash Commands

### Appending to a file

Write a bash one-liner that writes the current date to the file `~/tmp/date.txt` every 5 seconds. Create the directory if it doesn't exist yet. A new date does not overwrite the existing date in the file but appends it.

<details><summary>Show Solution</summary>
<p>

```
if [ ! -d ~/tmp ]; then mkdir -p ~/tmp; fi; while true; do echo $(date) >> ~/tmp/date.txt; sleep 5; done;
```

</p>
</details>

### Printing a counter to the console

Write a bash one-liner that defines a variable `counter` with the initial value 0. Increment the variable every second and print out its value to the console.

<details><summary>Show Solution</summary>
<p>

```
counter=0; while true; do counter=$((counter+1)); echo "$counter"; sleep 1; done;
```

</p>
</details>

### Creating a random number

Write a bash one-liner that defines a variable with a value between 1 and 100. In a loop, print the value of the variable if it is less than 50. Break the loop if the value is more or equal to 50 and print out the message "END: $value".

<details><summary>Show Solution</summary>
<p>

```
while true; do random=$(((RANDOM % 100) + 1)); if [ $random -le 50 ]; then echo "$random"; else echo "END: $random"; break; fi; sleep 1; done;
```

</p>
</details>

## Kubernetes Object Information

### Finding specific annotations

Print the sourrounding 10 lines of Pod description for any existing Pod with the annotation `author=John Doe`.

<details><summary>Show Solution</summary>
<p>

```
kubectl describe pods | grep -C 10 "author=John Doe"
```

</p>
</details>

### Finding all labels

Print labels for all Pods and determine their Pod names. Render the output in YAML format.

<details><summary>Show Solution</summary>
<p>

```
kubectl get pods -o yaml | grep -C 5 labels:
```

</p>
</details>