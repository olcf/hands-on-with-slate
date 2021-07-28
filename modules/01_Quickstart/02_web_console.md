# Web Console

The web UI for OpenShift is available from all of ORNL (you should be
able to reach it from your laptop on ORNL WiFi as well as the VPN).

## Exercise: Logging in with the web console

|Cluster|URL|
|---|---|
|Marble (Moderate Production cluster with access to Summit/Alpine)|[https://marble.ccs.ornl.gov](https://marble.ccs.ornl.gov)|
|Onyx (Open Production Cluster with access to Wolf)|[https://onyx.ccs.ornl.gov](https://onyx.ccs.ornl.gov)|

After logging in to the web console, you'll be on a Projects page.

## Exercise: Explore the Adminstrator and Developer Perspectives

In the navigation bar on the left, at the top you should see either **Developer** or **Administrator**
which you can toggle to switch between.

The **Administrator** perspective gives you an overview of the various Kubernetes and cluster objects such as:
**Operators**, **Workloads**, **Networking**, **Storage**, **Builds**

The **Develper** perspective has an overview **Topology** which tries to organize resources in a Project in a more
understandable way as well as the catalog for adding new deployments.

[Next](03_deploy.md)
