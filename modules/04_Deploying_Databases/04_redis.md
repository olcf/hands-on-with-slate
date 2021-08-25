# Deploying Redis with Service Catalog

In this section we will deploy a Redis instance using a Slate Helm Chart.

## Exercise: Deploy Redis

From the `Developer Perspective`, click the `+Add` and scroll down until you find the `Helm Chart` option. Select `Helm Chart`. Then scroll and select the `Redis v0.2.0 Provided by Slate Helm Charts` offering. Click `Install Helm Chart`.

From the `Install Helm Chart` window, a release name may be provided to distinguish the deployment. Take some time to explore the configuration options presented. When ready, click `Install`.

After a time, the view will switch over to `Topology`. Hovering over the StatefulSet will show the status of the objects change from Pending to Running. Clicking on the StatefulSet will display the pod and service objects deployed in support of the application as well as a link to view the pod logs.

## Exercise: Connect to Redis Service

To verify the service is up and running, we will connect to the pod and use the redis-cli command locally within the pod:

```bash
$ oc get pods
NAME      READY   STATUS    RESTARTS   AGE
redis-0   1/1     Running   0          5m51s
$ oc exec -it redis-0 sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
/data $ redis-cli
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> SET my_key "Slate"
OK
127.0.0.1:6379> get my_key
"Slate"
127.0.0.1:6379> exit
/data $
```

## Exercise: Cleanup

When exploration of the resources and instance is complete, we can cleanup the instance from the command line with:

```bash
$ helm ls
NAME 	NAMESPACE   	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
redis	<<<my-namespace>>>	1       	2021-08-25 02:28:20.232034465 +0000 UTC	deployed	redis-0.2.0
$ helm uninstall redis
release "redis" uninstalled
```
