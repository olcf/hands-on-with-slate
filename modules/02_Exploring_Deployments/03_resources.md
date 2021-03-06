# Resource Quotas

Setting resource requirements for your workloads is always a good idea. It ensures your workload
has enough resources to run as expected, as well as guaranteeing the workload's resources. Without
these settings, these will run under a Quality of Service (QoS) class as "Best Effort" and will
have no guarantee of continuing to run in the cluster. There is a plenty of functionality around
setting resources and priority classes of workloads.

[Upstream documentation on container resoures](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)

## Exercise: Setting Resources

Setting resources is done by setting **limits** and **requests**. Think if limits as your maximum
and requests as your minimum settings to run the workload. Setting them to the same value
**guarantees** the resources being available.

```bash
$ oc set resources deploy/hello-world-cli -c=hello-world-cli --requests=cpu=10m,memory=48Mi --limits=cpu=10m,memory=48Mi
```

## Exercise: OOM Kill Demo

To show what happens when a pod tries to use more memory that its allocated, copy the YAML to test:

```bash
$ cat <<EOF | oc apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo
spec:
  containers:
  - name: memory-demo-ctr
    image: polinux/stress
    resources:
      limits:
        memory: "100Mi"
      requests:
        memory: "100Mi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
EOF
```

We can see in the cluster this is being OOMKilled because it is trying to allocate more memory
than the container limits:

```bash
$ oc get pod memory-demo -o go-template="{{range .status.containerStatuses}}{{.lastState.terminated.reason}}{{end}}"
OOMKilled
```

Clean up the pod:

```bash
$ oc delete pod memory-demo
```

[Next](04_scaling_and_upgrading.md)
