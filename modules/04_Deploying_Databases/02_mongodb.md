# Deploying MongoDB with Service Catalog

As discussed in Module 1, the Developer Perspective provides access to deploy resources using the Service Catalog from the [olcf/slate-helm-charts](https://github.com/olcf/slate-helm-charts) repository.

## Exercise: Deploy MongoDB

From the `Developer Perspective`, click the `+Add` and scroll down until you find the `Helm Chart` option. Select `Helm Chart`. Then scroll and select the `Mongodb v1.0.0 Provided by Slate Helm Charts` offering. A new window is presented that provides a brief introduction to what the chart could manage as well as a listing of parameters available for use. When ready, select `Install Helm Chart`.

At the top of the `Install Helm Chart` window, a release name of `mongodb` has already been filled. You can replace this to me more or less specific as desired. In the form view, expand: MongoDB Database -> MongoDB -> Spec.

There are three versions of MongoDB available for deployment with the KubeDB operator. Select version 4.4.6. The remaining `Spec` options we will leave as is for this deployment.

Below the `MongoDB` section, there is another section called `Stash` which provides for backup of a MongoDB instance. By default this chart has backups disabled. If backups were desired, one would toggle false to true for `Enable Backups` and then changing the remaining options as needed for the desired backup regimen.

When ready, click `Install`. THe view will switch to the `Topology`. Hovering over the StatefulSet will show the status of the objects change from Pending to Running. If you click on the `SS mongodb` representation, all of the resources associated with the StatefulSet will be displayed. Notice that there is a single pod with a status of `Running` along with a link next to it that says `View Logs`. When you click this link, you will be taken to the logs view for the pod where you should see successful connections and authentications from liveness probes that are checking to ensure the application is available.

## Exercise: oc Exploration

It is possible to explore aspects of the MongoDB deployment as well using the `oc` command. From a terminal:

```bash
$ oc get mongodb
NAME      VERSION   STATUS   AGE
mongodb   4.4.6     Ready    7m10s
```

When we retrieve all instances of mongodb running in our project, we should see one listed with the release name entered in the form view of the previous exercise. Describing the resource provides other useful information:

```bash
$ oc describe mongodb mongodb
Name:         mongodb
Labels:       app.kubernetes.io/instance=mongodb
              app.kubernetes.io/managed-by=Helm
              app.kubernetes.io/name=mongodb
              helm.sh/chart=mongodb-1.0.0
Annotations:  meta.helm.sh/release-name: mongodb
              meta.helm.sh/release-namespace: stf042-mongo
API Version:  kubedb.com/v1alpha2
Kind:         MongoDB
...
Spec:
  Auth Secret:
    Name:  mongodb-auth
  Pod Template:
    Controller:
    Metadata:
    Spec:
    ...
    Liveness Probe:
        Exec:
          Command:
            bash
            -c
            set -x; if [[ $(mongo admin --host=localhost  --username=$MONGO_INITDB_ROOT_USERNAME --password=$MONGO_INITDB_ROOT_PASSWORD --authenticationDatabase=admin --quiet --eval "db.adminCommand('ping').ok" ) -eq "1" ]]; then 
          exit 0
        fi
        exit 1
        Failure Threshold:  3
        Period Seconds:     10
        Success Threshold:  1
        Timeout Seconds:    5
      ...
      Resources:
        Limits:
          Cpu:     1
          Memory:  1Gi
        Requests:
          Cpu:               500m
          Memory:            512Mi
      Service Account Name:  mongodb
  Replicas:                  1
  Ssl Mode:                  disabled
  Storage:
    Access Modes:
      ReadWriteOnce
    Resources:
      Requests:
        Storage:         1Gi
    Storage Class Name:  netapp-nfs
  Storage Engine:        wiredTiger
  Storage Type:          Durable
  Termination Policy:    WipeOut
  Version:               4.4.6
  ...
```

related to the resource deployed by the KubeDB operator such as the Liveness probe command observed earlier, the resources currently assigned to the pods, and the backend storage assigned to the instance.

## Exercise: Connect to MongoDB Service

We can test the application by retrieving the credentials for the MongoDB instance and then exec into the pod itself:

```bash
$ oc get mongodb
NAME      VERSION   STATUS   AGE
mongodb   4.2.3     Ready    36s
$ oc get pods
NAME        READY   STATUS    RESTARTS   AGE
mongodb-0   1/1     Running   0          105s
$ oc get secret mongodb-auth -o jsonpath='{.data.\username}' | base64 -D; echo
root
$ oc get secret mongodb-auth -o jsonpath='{.data.\password}' | base64 -D; echo
password
$ oc exec -it mongodb-0 sh
$ mongo admin
MongoDB shell version v4.2.3
connecting to: mongodb://127.0.0.1:27017/admin?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("ddbc66b3-281b-4bdb-af5a-2adbd434919a") }
MongoDB server version: 4.2.3
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
	http://docs.mongodb.org/
Questions? Try the support group
	http://groups.google.com/group/mongodb-user
> db.auth("root","password")
1
> show users
{
	"_id" : "admin.root",
	"userId" : UUID("cc51c3fa-fff3-450e-9cb9-48dd1f984ebf"),
	"user" : "root",
	"db" : "admin",
	"roles" : [
		{
			"role" : "root",
			"db" : "admin"
		}
	],
	"mechanisms" : [
		"SCRAM-SHA-1",
		"SCRAM-SHA-256"
	]
}
> exit
bye
$ exit
```

## Exercise: Cleanup

When exploration of the instance is complete, we can cleanup the instance from the command line with:

```bash
$ helm ls
NAME   	NAMESPACE   	REVISION	UPDATED                                	STATUS  	CHART        	APP VERSION
mongodb	<<<my-namespace>>>	1       	2021-08-25 01:41:16.392732032 +0000 UTC	deployed	mongodb-1.0.0
$ helm uninstall mongodb
release "mongodb" uninstalled
```

After a time, all resources should be removed and the `Topology` view will no longer display the MongoDB instance.

[Next](03_mysql.md)
