# Scaling and Upgrading the Deployment

In this section we will learn how to scale a Deployment up and run an upgrade on the Deployment
to update the image tag.

## Exercise: Exposing the Deployment

First we will expose the Deployment with a service that points to our workload. Think of this
as adding an internal DNS entry so other workloads in the cluster can reach the endpoint:

```bash
$ oc expose deployment hello-world-cli --name hello-world-cli-service --type ClusterIP --protocol TCP --port 8080 --target-port 8080
```

## Exercise: Start Sending Requests

Next we will run a pod another terminal window that will send endless requests to the service
endpoint to show load balancing:

```bash
$ oc run -i --tty send-requests --rm --image=busybox --restart=Never --requests=cpu=50m,memory=64Mi --limits=cpu=50m,memory=64Mi -- /bin/sh -c "while sleep 1; do wget -q -O- http://hello-world-cli-service:8080; done"
```

Leave this running in another terminal for observation.

## Exercise: Scale the Deployment

Observing the output of the terminal, you can see the hostname and version of the pod responding.
Now lets scale the deployment to 5 replicas and watch the output loadbalance between the pods:

```bash
$ oc scale deploy/hello-world-cli --replicas=5
```

To set Kubernetes to use this feature automatically based on load, see the
[Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/).

## Exercise: Upgrading the Deployment

When scaled with multiple replicas, a deployment upgrade (based on its upgrade strategy) will
only take down a percentage of pods at a time an replace them with a new image to prevent loss
of service. To see this, lets upgrade out deployment image to version 2.0:

```bash
$ oc set image deploy/hello-world-cli hello-world-cli=gcr.io/google-samples/hello-app:2.0
```

From here, you can see new pods come into existance while some are being deleted. In the terminal
sending wget requests to the service, you can observe traffic being a mix of v1.0 and v2.0 pods
until the deployment rollout is complete.

## Exercise: Cleanup

Clean up the resources since they aren't being used anymore.

Delete the Service and Deployment:

> Note: We are using the shortened names for Service (svc) and Deployment (deploy)

```bash
$ oc delete svc/hello-world-cli-service
$ oc delete deploy/hello-world-cli
```
