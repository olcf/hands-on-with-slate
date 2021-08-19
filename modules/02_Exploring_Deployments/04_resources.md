# Resource Quotas

Setting resource requirements for your workloads is always a good idea. It ensures your workload has enough resources to run as 
expected, as well as guaranteeing the workload's resources. Without these settings, these will run under a Quality of Service (QoS) class 
as "Best Effort" and will have no guarentee of continuing to run in the cluster. There is a plenty of functionality around setting resources 
and priority classes of workloads, for more information please click [here](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/).

## Exercise: Setting Resources

Setting resources is done by setting **limits** and **requests**. Think if limits as your maximum and requests as your minimum settings 
to run the workload. Setting them to the same value **guarantees** the resources being available. Lets do this now:
```
$ oc set resources deploy/hello-world-cli -c=hello-world-cli --requests=cpu=50m,memory=48Mi --limits=cpu=50m,memory=48Mi
```

## OOM Kill Demo?
Now lets expose the deployment with a service that points to your workload. Think of this as adding an internal DNS entry so other workloads 
in the cluster can reach the endpoint:
```
oc expose deployment hello-world-cli --name hello-world-cli-service --type ClusterIP --protocol TCP --port 8080 --target-port 8080
```

Next lets run a pod that will send endless requests to the service endpoint to generate load:
```
oc run -i --tty load-generator --rm --image=busybox --restart=Never --requests=cpu=50m,memory=48Mi --limits=cpu=50m,memory=48Mi -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://hello-world-cli-service:8080; done"
```

[Next](05_scaling_and_upgrading.md)