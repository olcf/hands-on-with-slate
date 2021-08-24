# Deploy a Complex Application: Airflow

[Apache Airflow](https://airflow.apache.org/) is a platform to monitor and schedule
workflows. It is written in Python and the workflows are written as directed graphs
in Python as well.

It is a good example of deploying a complex application because it is designed to scale
which means while the application could all run in a single pod we will be deploying multiple
layers including database and caching tiers as well as separating out the application into
a web frontend and scheduler backend with workers all as separate deployments.

To deploy this stack we will leverage the Airflow project's supported [Helm chart](https://airflow.apache.org/docs/helm-chart/stable/index.html).

## Exercise: Deploy PostgreSQL Database

First we need to deploy a PostgreSQL database for storing data for the application. We will
use the Slate Helm chart repository which is the same repository that is used to populate
the service catalog on the cluster web interface.

We can add the repository just like before and then install the postgresql chart with
the default options.

```bash
$ helm repo add olcf-slate https://olcf.github.io/slate-helm-charts
"olcf-slate" has been added to your repositories

$ helm install postgresql olcf-slate/postgresql
```

This will create and manage a PostgreSQL database using KubeDB Enterprise which is managed
by the NCCS Platforms group and makes it easier to manage databases in Kubernetes.

The database installation will create a Secret that contains the username and password
to log into the database which we will need to configure Airflow. We can dump the username
and password with `oc extract` and it will create a secret of the form: RELEASENAME-auth
which in this case our release is named **postgresql**.

```bash
$  oc extract secret/postgresql-auth --to=-
# password
password
# username
postgres
```

Finally, we can check that the database is up and ready to accept connections by checking
the StatefulSet that it deploys:

```bash
$ oc get pods -l app.kubernetes.io/instance=postgresql 
NAME           READY   STATUS    RESTARTS   AGE
postgresql-0   2/2     Running   0          2m
```

Looking at the output, **2/2** and **Running** means we know that the pod is running and
accepting connections.

## Exercise: Deploy Airflow

In order to deploy Airflow, the chart needs to be installed with the UID and GID that the
pods will run as. In Slate, all workloads run as regular users and cannot run as root
in the container for security reasons. There has been a big push in the community to update
container images and charts to allow them to run as any specified UID and GID which the Airflow
chart allows with the `uid` and `gid` values for the chart.

We can get the UID and GID that Airflow needs to run as by using `oc describe` on the Kubernetes
Namespace that we are deploying into. 

In the example I am describing the **stf042** namespace:

```bash
$ oc describe ns stf042
Name:         stf042
Annotations:  ccs.ornl.gov/defaultNamespace: false
              ccs.ornl.gov/runAsGroup: 28763
              ccs.ornl.gov/runAsUser: 16717
              ccs.ornl.gov/runAsUsername: stf042_auser
              ccs.ornl.gov/supplementalGroups: 1099,2001,24121,26118,26694,27493,28763,28767
...
Status:       Active
...
```

As we can see in the Annotations for the namespace, we are running as UID **16717** and GID **28763**
which is the UID and GID of the STF042 automation user.

Next we need to add the Apache Airflow chart repository:

```bash
$ helm repo add apache-airflow https://airflow.apache.org
"apache-airflow" has been added to your repositories
```

We use a values file in this installation of Airflow. The `values.yaml` is used to set a
number of default options specific to using Airflow in Slate for the purposes of this tutorial.

> We can get the default values.yaml of the Airflow chart (which is long) by
running: `helm show values apache-airflow/airflow` which is how `airflow-values.yaml` was initially created.

Now we can install the Airflow chart using the values file in this tutorial module and setting
our values:

| Key | Value |
| --- | ----- |
| UID | runAsUser annotation on Namespace |
| GID | runAsGroup annotation on Namespace |
| SOME-HOST | A unique hostname without dots which we will use to access the web interface of Airflow |
| CLUSTER | marble or onyx depending on which cluster you are installing Airflow on |
| DB-USER | Username created during the PostgreSQL installation |
| DB-PASSWORD | Password created during the PostgreSQL installation |

```bash
$ helm install airflow apache-airflow/airflow \
  --values airflow-values.yaml \
  --set uid=UID \
  --set gid=GID \
  --set ingress.web.host=SOME-HOST.apps.CLUSTER.ccs.ornl.gov \
  --set data.metadataConnection.user=DB-USER \
  --set "data.metadataConnection.pass=DB-PASSWORD" \
  --set data.metadataConnection.host=postgresql
```

> This `helm install` command will take a long time to complete.

Once the install completes, let's check the pods:

```bash
$ oc get pods -n stf042
NAME                                 READY   STATUS    RESTARTS   AGE
airflow-flower-68f99f89cf-vt5tf      1/1     Running   0          113s
airflow-redis-0                      1/1     Running   0          112s
airflow-scheduler-5f66869649-bbvnz   2/2     Running   0          113s
airflow-webserver-95cc46cb6-qr64p    0/1     Running   0          113s
airflow-worker-0                     2/2     Running   0          112s
postgresql-0                         2/2     Running   0          26m
```

We see that we still have the webserver not ready **0/1** so we can wait for all pods:

```bash
$ oc wait pods --for=condition=Ready --all -n stf042
pod/airflow-flower-68f99f89cf-vt5tf condition met
pod/airflow-redis-0 condition met
pod/airflow-scheduler-5f66869649-bbvnz condition met
pod/airflow-webserver-95cc46cb6-qr64p condition met
pod/airflow-worker-0 condition met
pod/postgresql-0 condition met
```

Once the `oc wait` command returns we should be able to open a web browser and access
`https://SOME-HOST.apps.CLUSTER.ccs.ornl.gov` which will first prompt for your OLCF credentials
and then allow you to access the Airflow UI which you can log in with the default username/password
of **admin/admin**.

## Exercise: Test Out Airflow

At this point you can enable and test running the example DAGs that were pre-populated as part of the
demo.

## Exercise: Clean up

Clean up the Airflow installation and the database

```bash
$ helm delete airflow
release "airflow" uninstalled

$ helm delete postgresql
release "postgresql" uninstalled
```

Clean up all PersistentVolumeClaims which do not get removed when deleting the releases

```bash
$ oc get pvc
NAME                       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-postgresql-0          Bound    pvc-61e7b700-1bdb-44bf-813d-4d3d8eb8a5ad   1Gi        RWO            netapp-block   17m
logs-airflow-worker-0      Bound    pvc-ec1e165d-d286-483b-91cd-fa09d8aabab6   1Gi        RWO            netapp-nfs     16m
redis-db-airflow-redis-0   Bound    pvc-f2656e21-9517-4c51-900d-306b331870e5   1Gi        RWO            netapp-nfs     16m

$ oc delete pvc data-postgresql-0 logs-airflow-worker-0 redis-db-airflow-redis-0
persistentvolumeclaim "data-postgresql-0" deleted
persistentvolumeclaim "logs-airflow-worker-0" deleted
persistentvolumeclaim "redis-db-airflow-redis-0" deleted
```
