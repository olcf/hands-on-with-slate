# Debugging Container With oc debug

Kubernetes is doing its part but for some reason the code inside the application cannot run. The log messages are not helpful. What would be really helpful would be if you could get into that pod and poke around a little bit...

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

You will notice that the container does not start and the Pod is in an errored state:

```bash
oc get pods
```

to see that there are `0/1` containers ready and that the pod is in an `Error` state.

* If you wait long enough the pod will go into a `CrashLoopBackOff` state. Kubernetes does not really know anything it can do to get your code working other than just restarting the container. So it tries that. And when that does not work enough times the state will go to `CrashLoopBackOff` meaning it is going to back off how often it restarts your container. 

The first step when the application code is failing is to get the logs:

```bash
oc logs test-pod
```

Now, for this exercise the logs make the error pretty clear and you could solve the problem with just them. But lets pretend the application does not have the best logging.

In a scenario like that we would want to open a debug container. These containers are ephemeral and will only exist for the duration of there being an active shell.

These containers are the same as your container with the key difference being their start command is just putting you in the shell instead of the command that you defined in your Pod spec.

To launch a debug container run:

```bash
oc debug test-pod
```

this will put us in a shell. 

Alright, now lets try running through the start commands we listed. `/bin/sh` seems good. So does the first arg `echo`. `catt`, however, gives a command not found error. You do not remember exactly the command that you were looking for but you do know it begins with a "ca" so you run:

```bash
ls /bin | grep ca
```

And, oh yeah, it is `cat` not `catt`.

You fix the Pod spec:

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

Delete the crashing pod

```bash
oc delete pod test-pod
```

and apply the fixed spec:

```bash
oc apply -f test-pod.yaml
```

### Recap

* If you application is failing check the logs with `oc logs {POD NAME}`
* If the logs are not sufficient, get a shell in the pod and start poking around with `oc debug {POD NAME}`
