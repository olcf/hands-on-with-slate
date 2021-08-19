# Container Monitoring

Everything that gets logged to stderr or stdout is captured by Kubernetes

## Exercise: Get Pod Logs

Run the following command to get the log output for your deployment:
```
$ oc logs deploy/hello-world-cli
2021/08/19 17:41:13 Server listening on port 8080
2021/08/19 17:41:28 Serving request: /
2021/08/19 17:41:32 Serving request: /
2021/08/19 17:41:33 Serving request: /
```

## Exercise: Port-Forward

Port forwarding allows you to forward a connection to the pod. Run the following and go to localhost:8080 in your browser:
```
oc port-forward deploy/hello-world-cli 8080:8080
```
## Exercise: Explore Metrics

Under **Developer** perspective on the OpenShift console web interface, click **Monitoring** to view resource usage of your workload.

[Next](03_container_shell.md)
