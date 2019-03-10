# Bonus Exercises

## Bash commands

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

## Grep Kubernetes object information

Print the sourrounding 10 lines of Pod description for any existing Pod with the annotation `author=John Doe`.

<details><summary>Show Solution</summary>
<p>

```
kubectl describe pods | grep -C 10 "author=John Doe"
```

</p>
</details>