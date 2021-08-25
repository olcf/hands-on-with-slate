# Deploying MySQL with Service Catalog

In this section we will deploy a MySQL instance using a Slate Helm Chart.

## Exercise: Deploy MySQL

From the `Developer Perspective`, click the `+Add` and scroll down until you find the `Helm Chart` option. Select `Helm Chart`. Then scroll and select the `Mysql v1.0.0 Provided by Slate Helm Charts` offering. A new window is presented that provides a brief introduction to what the chart could manage as well as a listing of parameters available for use. When ready, select `Install Helm Chart`.

On the `Install Helm Chart` window, fill in an release name for this instance. This chart deploys three components: a MySQL database, a phpMyAdmin instance fronting the MySQL database, and a Stash object that performs daily backups of the MySQL instance. Expand `MySQL Database` to view the customization options for the three components deployed with this chart. This chart will allow for the deployment of two different versions of MySQL (5.7.33 or 8.0.23). Select `5.7.33-v1` and leave the remaining options for the Mysql spec section as is. In the phpMyAdmin leave the instance enabled. For the Stash section, toggle `Enable Backups` to false. Click `Install` to deploy this instance.

After a time, the view will switch over to `Topology`. Hovering over the status of the objects will show the status of objects change from Pending to Running. There are multiple resources deployed for the MySQL and phpMyAdmin components: the phpMyAdmin deployment resource consisting of one pod, two configMap resources for phpMyAdmin, a PVC for phpMyAdmin, a Service and Route for phpMyAdmin, and then MySQL instance itself. Clicking on the Route should take you to the Route Details screen where a link is presented to the location of the phpMyAdmin instance. Clicking on the location link should prompt you to authenticate with OpenShift and then display the phpMyAdmin interface tied into the database that was created.

## Exercise: oc Exploration

From the command line, use the `oc` command to explore various resources deployed:

```bash
$ oc get mysql
NAME    VERSION     STATUS   AGE
mysql   5.7.33-v1   Ready    6m9s
$ oc get route
NAME               HOST/PORT                                                 PATH   SERVICES           PORT    TERMINATION     WILDCARD
mysql-phpmyadmin   mysql-phpmyadmin-<<<my-namespace>>>.apps.marble.ccs.ornl.gov          mysql-phpmyadmin   <all>   edge/Redirect   None
$ oc get svc
NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
mysql              ClusterIP   172.25.167.22   <none>        3306/TCP   6m55s
mysql-phpmyadmin   ClusterIP   172.25.252.84   <none>        8080/TCP   6m56s
mysql-pods         ClusterIP   None            <none>        3306/TCP   6m55s
$ oc get pods
NAME                                READY   STATUS    RESTARTS   AGE
mysql-0                             1/1     Running   0          7m17s
mysql-phpmyadmin-654fff6b6f-65jfm   1/1     Running   0          7m18s
$ oc get pvc
NAME                       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-mysql-0               Bound    pvc-98d6bb64-5bda-4e1e-bb65-cd8e409fa1b2   1Gi        RWO            netapp-nfs     12m
mysql-phpmyadmin-tempdir   Bound    pvc-37948cfb-47f6-4afc-8960-db95b5a5b7c1   50Mi       RWX            netapp-nfs     12m
```

Notice that there are two services for the MySQL database listed as well as a PersistentVolumeClaim that were not in the `Topology` view in the web console. Also, there is a pod named `mysql-0` which indicates that there should be a StatefulSet present, but that also was not listed in the `Topology` view. If we run the command:

```bash
$ oc get StatefulSet -o wide
NAME    READY   AGE     CONTAINERS   IMAGES
mysql   1/1     9m25s   mysql        mysql:5.7.33
```

we discover more information about the StatefulSet. The `Topology` view is limited for KubeDB database instances in that it hides most of the created resources with the `MSQL mysql` resource. This should be kept in mind when attempting to use the `Developer Perspective` for troubleshooting. If instead the command line or the `Administrator Perspective` is used, a more complete picture of the objects deployed along with their properties may be developed.

## Exercise: Cleanup

When exploration of the MySQL and phpMyAdmin instance is complete, we can cleanup the instance from the command line with:

```bash
$ helm ls
NAME 	NAMESPACE   	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
mysql	<<<my-namespace>>>	1       	2021-08-25 02:06:43.617108422 +0000 UTC	deployed	mysql-1.0.0

[Next](04_redis.md)
