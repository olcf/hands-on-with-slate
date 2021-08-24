# Introduction to Helm

[Helm](https://helm.sh/) is a package manager for Kubernetes. It has three main concepts:

| Concept | Definition |
|---|---|
| Charts | A chart contains the resource definitions needed to run an application |
| Repository | A repository holds a one or more charts and is a central location to download and provide updated charts |
| Release | A Release is a an instance of a chart in a cluster |

From the upstream doc: Helm installs charts into Kubernetes, creating a new release for each installation. And to find new charts, you can search Helm chart repositories.

See [upstream documentation](https://helm.sh/docs/intro/using_helm/) for more information

## Exercise: Download Helm

Helm only consists of a client binary and can be downloaded from:

* The official releases page: https://github.com/helm/helm/releases
* OS Package managers: https://helm.sh/docs/intro/install/

## Exercise: Add Chart Repository

In order to install charts first we need to add a repository definition.

> There are no "official" repositories from the Kubernetes community since the old *stable*
> repository was retired.

The community has moved to a distributed model rather than a monolitic repository for all
applications. Applications that have official support for Kubernetes and use Helm will generally
publish their own repository with their charts.

To get started quickly we will use the Bitnami chart repository. Bitnami has made a lot of
charts for various popular applications that do not have their own helm charts.

```bash
$ helm repo add bitnami https://charts.bitnami.com/bitnami
```

## Exercise: Install a Chart

> If you are already logged into a cluster, the `helm` client will reuse your login credentials
> otherwise first log into the cluster with `oc login`

We will use the standard Nginx chart from Bitnami but we will need to set a few options so that it works with Slate.

The options or **values** of a chart are unique to each chart and are used to change how the
resources are created. We can inspect the entire YAML-formatted values file with
`helm show values` but the Bitnami chart values is enormous so in this case it may be easier
to go to the [ArtifactHub for bitnami/nginx](https://artifacthub.io/packages/helm/bitnami/nginx)
which has better formatting and description of all of the options for the chart.

```bash
$ helm show values bitnami/nginx
## @section Global parameters
## Global Docker image parameters
## Please, note that this will override the image parameters, including dependencies, configured to use the global value
## Current available global Docker image parameters: imageRegistry, imagePullSecrets and storageClass

## @param global.imageRegistry Global Docker image registry
## @param global.imagePullSecrets Global Docker registry secret names as an array
##
global:
  imageRegistry: ""
...
```

We will install the nginx chart with some options described below. Change `SOME-HOST` to a
unique value for a hostname and change `CLUSTER` to the cluster name such as **marble**
or **onyx** which will be exposed on that cluster's load balancers.

```bash
$ helm install test bitnami/nginx \
  --set ingress.enabled=true \
  --set ingress.hostname=SOME-HOST.apps.CLUSTER.ccs.ornl.gov \
  --set-string "commonLabels.ccs\.ornl\.gov/externalRoute=true"
NAME: test
...
```

Looking at each option:

`--set ingress.enabled=true`

Create the Ingress object which will enable routing our HTTPS traffic from the cluster load balancers to our application

`--set ingress.hostname=SOME-HOST.apps.CLUSTER.ccs.ornl.gov`

The Ingress object does not define a hostname by default (unlike the OpenShift-specific Route)
so we need to define one ourselves. By default, we have DNS set up to forward all .apps.cluster.ccs.ornl.gov to the cluster load balancers so as long as it is a unique hostname that has not already been taken by another project it can be used.

`--set-string "commonLabels.ccs\.ornl\.gov/externalRoute=true"`

This option is a complicated escape of our `ccs.ornl.gov/externalRoute` label that ensures that
the Ingress will be exposed on the external cluster loadbalancers instead of just the ORNL-internal ones.

Once the chart has been deployed we can wait for the deployment to be ready:

```bash
$ oc rollout status deployment/test-nginx
deployment "test-nginx" successfully rolled out
```

Once it is deployed and the pod is running we can access the exposed route by opening `https://SOME-HOST.apps.CLUSTER.ccs.ornl.gov` in a web browser.

## Exercise: Uninstalling a Release

One of the nice things about Helm is that the Release keeps track of all of the resources
associated with the deployment and when we uninstall the release everything gets deleted.

```bash
$ helm uninstall test
release "test" uninstalled
````

[Next](02_simple_application.md)
