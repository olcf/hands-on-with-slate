# Monitoring Deployments

## Exercise: Get Pod Logs

Everything that gets logged to stderr or stdout is captured by Kubernetes. Run the following
command to get the log output for your Deployment:

```bash
$ oc logs deploy/hello-world-cli
2021/08/19 17:41:13 Server listening on port 8080
2021/08/19 17:41:28 Serving request: /
2021/08/19 17:41:32 Serving request: /
2021/08/19 17:41:33 Serving request: /
```

## Exercise: Port-Forward

Port forwarding allows you to forward a connection to the pod. Run the following and go to
localhost:8080 in your browser:

```bash
$ oc port-forward deploy/hello-world-cli 8080:8080
```

## Exercise: Get a Shell

Sometimes you need to access the container in the cluster to debug, this can be done using
the **exec** function:

```bash
$ oc exec -it deploy/hello-world-cli -- /bin/sh
~ $
~ $ exit
```

## Exercise: Explore Metrics

Under **Developer** perspective on the OpenShift console web interface, click **Monitoring**
to view resource usage of your workload.

[Next](03_resources.md)
