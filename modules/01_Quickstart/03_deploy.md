# Deploy a Docker Image

We will deploy a docker image from DockerHub using the **Add +** catalog interface in the web console

## Exercise: Deploy with Catalog

Switch to the **Develper** perspective by changing the drop down at the top of the left menu bar.

> The Developer perspective is designed to make it easy to deploy and see an entire application in the cluster.

Click the **+Add** button on the left menu bar and choose **Helm Chart**.

> Helm is a tool for deploying entire applications onto Kubernetes. It handles the YAML configuration through templates.
> The OLCF maintains a set of charts for Helm for helping users deploy common pieces of infrastructure so they can focus
> on their application: [olcf/slate-helm-charts](https://github.com/olcf/slate-helm-charts)

Select the Deploy Image chart which we will use to deploy a generic container image onto Slate.

Fill out the form and click **Install** at the bottom:

- Release Name: **hello-world**
- Image: **openshift/hello-openshift:latest**

You should be taken back to the Topology view but now there is a box that represents the application that has been deployed
with our Helm chart. If you click on the circle or the label with the **D** then a sidebar will expand to show you the details
of the **Deployment** Kubernetes object which has been created. From here you can click through to the running **Pod** which
is where the container image is acutally running and perform actions such as getting logs and execing into the container with
a shell.

We can tell that the process was started in the container because the Pod is showing a **Running** status.

[Next](04_route.md)
