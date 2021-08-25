# Debugging Container With oc debug

## Exercise: Crash Loop Pod

When a container crashes Kubernetes will restart that container in hopes of fixing the problem. If the container keeps crashing after subsequent restarts Kubernetes will back off on how often it restarts that container. When this happens the Pod State is changed to `CrashLoopBackoff`.


To start, apply the following Pod spec:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  tolerations:
  - operator: "Exists"
  containers:
    - name: test-pod
      image: "centos:7"
      command: ["/bin/sh","-c"]
      args: ["echo 'Hello World!'; catt"]
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File
      tty: true
      stdin: true
  dnsPolicy: ClusterFirst
  terminationGracePeriodSeconds: 5
```

When this Pod is created we can immediately see that the Pod is in an `Error` state by running:

```bash
oc get pods
```

From the overview of Pods we can see that the Pod is in the `Error` state and zero out of one containers are ready `0/1`. In previous modules we used `oc describe` or `oc get events` to debug our containers but for some problems those tools will not be enough.

When there are no events under `oc get events` that seem errant and there are no scheduling problems you can start to assume that something is incorrect inside the Pod.

The first step when this code is suspected is to get the logs from the Pod:

```bash
oc logs test-pod -c test-pod
```
`oc logs` will get the logs from the container, specified after the `-c` flag, from the Pod specified after `oc logs`. The command is of the form:

```bash
oc logs {POD_NAME} -c {CONTAINER_NAME}
```

From the logs you can see that the command `catt` does not exist. Looking back at our Pod spec we see that we have accidentally typed `catt` when we meant `cat.

If we correct this:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  tolerations:
  - operator: "Exists"
  containers:
    - name: test-pod
      image: "centos:7"
      command: ["/bin/sh","-c"]
      args: ["echo 'Hello World!'; cat"]
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File
      tty: true
      stdin: true
  dnsPolicy: ClusterFirst
  terminationGracePeriodSeconds: 5
```

Then the Pod will run.

## Exercise: Crash Looping Pod Without Logs

There can be scenarios where there are no Events, no Scheduling problems and not logs. In a situation like this use the `oc debug` command to attach an ephemeral container to the problematic container. This opens a shell in an environment like the one of the failing container and will allow you to poke around to see what is amiss. 

First, in case there are any old example Pods left run:

```bash
oc delete pod test-pod
```

Now create the following Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  tolerations:
  - operator: "Exists"
  containers:
    - name: test-pod
      image: "centos:7"
      command: ["/bin/sh","-c"]
      args: ["echo 'foo' > /bar.txt; cat"]
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File
      tty: true
      stdin: true
  dnsPolicy: ClusterFirst
  terminationGracePeriodSeconds: 5
```

```bash
oc apply -f test-pod.yaml
```

Like the previous debug Pod you will notice this Pod is in an Error state. We check the logs:

```bash
oc logs test-pod -c test-pod
```

and see a Permissions error.

What would really help us here is to poke around inside the Container and check the permissions of things.

When you create a Pod you define a command for the containers in that pod to run to start. Sometimes that command will fail. `oc debug` allows you to start the container with the command just being a shell so you can manually try running the commands that are failing.

So we will open a debug shell in our Pod:

```bash
oc debug test-pod
```

This will place you in a shell and will also print out what the command that the original container tried to start with was. Lets run that command from within our debug shell:

```bash
/bin/sh -c echo foo > /bar.txt
```

We see the Permission denied again. Some basic Linux debugging steps now come in handy:

```bash
ls -l /
whoami
```
And you can see that you do not have ownership anywhere to write your file.

This is a great example of when to use an `emptyDir`. And `emptyDir` is, as the name would suggest, an empty directory that will live for as long as the pod. It will only exist so long as the Pod is scheduled to the Node and as such should only be used for scratch space. 

The `emptyDir` will be world writeable. We add an `emptyDir` to the `/emptyDir` mount point and change our command to write there instead:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  tolerations:
  - operator: "Exists"
  containers:
    - name: test-pod
      image: "centos:7"
      command: ["/bin/sh","-c"]
      args: ["echo 'foo' > /emptyDir/bar.txt; cat"]
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File
      tty: true
      stdin: true
      volumeMounts:
      - mountPath: /emptyDir
        name: our-empty-dir
  volumes:
    - name: our-empty-dir
      emptyDir: {}
  dnsPolicy: ClusterFirst
  terminationGracePeriodSeconds: 5
```
> cat is added as the final command because a blank cat command will just run indefinitely. This will keep our Pod `Running` and prevent it from `Completing`.

### Recap

* If you application is failing check the logs with `oc logs {POD NAME}`
* If the logs are not sufficient, get a shell in the pod and start poking around with `oc debug {POD NAME}`
* You can use an `emptyDir` as scratch space if you need to write a file
