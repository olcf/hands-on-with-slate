# Container Status

Imagine you have just made some changes to a working Pod spec and all the sudden there is no Pod.....

To simulate this, apply the following Pod Spec:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  nodeSelector:
    kubernetes.io/hostname: maarble1.ccs.ornl.gov
  containers:
    - name: test-pod
      image: "cents:7"
      command: ["/bin/sh","-c"]
      args: ["echo 'Hello World!'; cat"]
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File
      tty: true
      stdin: true
  dnsPolicy: ClusterFirst
  terminationGracePeriodSeconds: 5
```

If the above YAML is saved as `pod.yaml` we would run:

```bash
oc apply -f pod.yaml
```

Now if you run

```bash
oc get pods
```

You will see that only `0/1` containers are ready and that the status of the Pod is `Pending`. Hmm, that is not enough information. To get more information run:

```bash
oc get events
```

This will show us events from Kubernetes itself. We know that the pod has not been scheduled yet, because it is `Pending` so we know that the issue is above the code in the Application and is instead in the way we are trying to configure our Pod.

You should see a `FailedScheduling` event for our Pod. That is because in the above spec we were specifying a node label that does not exist. We do not even need to specify any labels so lets remove that from the spec and then apply again. The updated Pod Spec to apply is:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
    - name: test-pod
      image: "cents:7"
      command: ["/bin/sh","-c"]
      args: ["echo 'Hello World!'; cat"]
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File
      tty: true
      stdin: true
  dnsPolicy: ClusterFirst
  terminationGracePeriodSeconds: 5
```

And lets get a clean slate:

```bash
oc delete pod test-pod
```

then:

```bash
oc apply -f pod.yaml
```

Alright now just run:

```bash
oc get pods
``` 

and if you are quick enough you will see the pod in Status `ContainerCreating`! Excellent! The Pod is scheduled, get pods again:

```bash
oc get pods
```

...to see that `0/1` containers are ready and that the new Status is `ErrImagePull`.

Back to Events!

```bash
oc get events
```

Here you can see the new Failed event:

```
Warning   Failed             pod/test-pod            Failed to pull image "cents:7": rpc error: code = Unknown desc = Error reading manifest 7 in docker.io/library/cents: errors:
denied: requested access to the resource is denied
```

Ohh, we have a typo in our image name. We were trying to pull `cents:7` when we meant to pull `centos:7`. Below is a Pod spec with both of our previous errors fixed.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
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

And again!

```bash
oc delete pod test-pod
```

```bash
oc apply -f pod.yaml
```

``bash
oc get pods
```

And......its running!


### Recap

Here we learned to always check our Container status if the application is not launching. We can check for problems above the code in our application by running:

```bash
oc get events
```

[Next](03_debug.md)
