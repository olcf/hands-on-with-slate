# Add Persistent Storage

Since the container image is ephemeral by design, we will use the on-demand
storage capabilities of Slate to provision storage that is persistent across
restarts of the Pods.

Slate has two primary means of adding persistent storage to a container deployment:

- Kubernetes-native on-demand storage using PersistentVolumeClaims
- Directly mounting the massively scalable center-wide parallel filesystems

This can be a tricky part of deploying containerized applications because you
have to be aware of where your application is writing data. Not all applications
are clear about where they expect to be able to write to on the filesystem and
which files are ephemeral (such as lock or temp files) and which are expected
to be persistent.

## Exercise: Adding Kubernetes-native Persistent Storage

Switching to the **Developer** perspective, we can add persistent storage to our existing
Helm chart deployment of Nginx.

From the left-hand navigation click on **Helm** and use the contextual three dots to Upgrade the
Helm Release for Nginx to add add persistence using the chart.

- Data Persistence:
  - Enabled: **True**
  - Size: **1Gi**
  - Storage Class: **netapp-nfs**
  - Mount path: **/usr/share/nginx/html/data**

The chart will create a PersistentVolumeClaim and Kubernetes will mount it at /usr/share/nginx/html/data.

### Exercise: Test Persistence

From the left-hand navigation click on **Topology** and then click on the blue circle or the `D hello-world`
label and then click on the Pod in the right-hand informational drawer. We are now viewing the details of a Pod that
was created as part of the Deployment of our hello-world application.

Click on **Terminal** and then in the shell:

> echo "hello from a persistent volume" > /usr/share/nginx/html/data/hello.html

Now click the **Actions** menu for the Pod and **Delete Pod** and wait for the new pod to be scheduled and
start running. Try to access the route URL (URL can be found under **Topology** and clicking on the deployment
and checking the bottom of the right-hand informational drawer) and appending `/data/hello.html` which should show
"hello from a persistent volume"

### Exercise: Show PersistentVolumeClaims

Using the OpenShift console under the **Administrator** perspective, there is an option
for Storage -> PersistentVolumeClaims which will show all storage that is provisioned
in the project.

## Exercise: Adding Writable Directories to Container Image

[Next](07_next_steps.md)
