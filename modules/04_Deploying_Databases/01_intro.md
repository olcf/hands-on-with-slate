# Introduction to Database Deployments

Unlike a `Deployment` object discussed in Module 2, database applications are typically deployed using a `StatefulSet` object that contains the database application along with a `PersistentVolumeClaim` (PVC) for the database store. As seen in [Module 3: Complex Airflow](../03_Deploying_with_Helm/03_complex_airflow.md), we leveraged KubeDB Enterprise to install and configure an instance of PostgreSQL 13. KubeDB provides multiple Custom Resource Definition (CRD) objects to define database types for use in the cluster alongside multiple operators to manage all lifecycle aspects of instance of a supported database. For example, KubeDB provides two CRDs for memcached:

```bash
$ oc get crd | grep kubedb | grep memcached
memcacheds.kubedb.com                                             2021-03-04T19:51:20Z
memcachedversions.catalog.kubedb.com                              2021-03-04T19:51:22Z
```

The `memcachedversions.catalog.kubedb.com` CRD provides a listing of versions supported by the operator:

```bash
$ oc get memcachedversions
NAME       VERSION   DB_IMAGE                    DEPRECATED   AGE
1.5.22     1.5.22    kubedb/memcached:1.5.22                  120d
1.5.4-v1   1.5.4     kubedb/memcached:1.5.4-v1                120d
```

while the `memcacheds.kubedb.com` CRD provides the ability to deploy an instance of memcached.

## Exercise: Deploy memcached

Within the Module 4 directory, there is a [code file](code/memcached-quickstart.yaml):

```bash
$ cat code/memcached-quickstart.yaml 
apiVersion: kubedb.com/v1alpha2
kind: Memcached
metadata:
  name: memcd-quickstart
spec:
  replicas: 1
  version: "1.5.4-v1"
  podTemplate:
    spec:
      resources:
        limits:
          cpu: 500m
          memory: 128Mi
        requests:
          cpu: 250m
          memory: 64Mi
  terminationPolicy: WipeOut
```

From the command line, we can create the KubeDB memcached deployment from this file:

```bash
$ oc create -f code/memcached-quickstart.yaml
memcached.kubedb.com/memcd-quickstart created
```

and verify the deployment:

```bash
$ oc get pods
NAME                 READY   STATUS    RESTARTS   AGE
memcd-quickstart-0   1/1     Running   0          7s
$ oc get svc
NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
memcd-quickstart        ClusterIP   172.25.235.151   <none>        11211/TCP   12s
memcd-quickstart-pods   ClusterIP   None             <none>        11211/TCP   12s
```

For KubeDB deployments, the pod status of `Running` indicates that the pods are running; however, the application may not yet be ready to receive requests. To verify if the application is available, retrieve the object type itself:

```bash
 oc get memcached
NAME               VERSION    STATUS   AGE
memcd-quickstart   1.5.4-v1   Ready    118s
```

The `Ready` status indicates that this memcached instance is ready for use and responding correctly. A status of `Provisioning` would indicate that the pods are still being spun up and initialized.

## Exercise: Connecting to memcached

We can connect to the memcached instance to test functionality. First, in one terminal we create a port forward to either the service of pod:

```bash
$ oc port-forward pods/memcd-quickstart-0 11211:11211
Forwarding from 127.0.0.1:11211 -> 11211
Forwarding from [::1]:11211 -> 11211
```

and then in a second terminal, we can test the application:

```bash
$ telnet 127.0.0.1 11211
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
set my_key 0 2592000 1
2
STORED
get my_key
VALUE my_key 0 1
2
END
quit
Connection closed by foreign host.
```

## Exercise: memcached Removal

Cleanup of a memcached instance that is no longer needed is easily accomplished:

```bash
$ oc get memcached
NAME               VERSION    STATUS   AGE
memcd-quickstart   1.5.4-v1   Ready    6m1s
$ oc delete memcached memcd-quickstart
memcached.kubedb.com "memcd-quickstart" deleted
$ oc get pods
No resources found in <<<my-namespace>>> namespace.
$ oc get svc
No resources found in <<<my-namespace>>> namespace.
```

## References

* [https://kubernetes.io/docs/tasks/run-application/run-single-instance-stateful-application/](https://kubernetes.io/docs/tasks/run-application/run-single-instance-stateful-application/)
* [https://docs.openshift.com/container-platform/4.7/storage/understanding-persistent-storage.html](https://docs.openshift.com/container-platform/4.7/storage/understanding-persistent-storage.html)
* [https://kubedb.com/docs/v2021.08.23/guides/](https://kubedb.com/docs/v2021.08.23/guides/)

[Next](02_mongodb.md)
