# Deploy a Simple Application

We will be deploying n8n which is a popular workflow system

## Exercise: Add Slate Helm Chart repository

First we will add the Slate Helm chart repository with `helm repo add`. This is the same
repository that is used to create the Slate Service Catalog in the developer web interface.

```bash
$ helm repo add olcf-slate https://olcf.github.io/slate-helm-charts
"olcf-slate" has been added to your repositories
```

We can check that this repository has been added by searching

```bash
$ helm search repo olcf-slate
NAME                   	CHART VERSION	APP VERSION	DESCRIPTION
olcf-slate/deploy-image	0.4.1        	           	Deploy a container image
...
```

## Exercise: Deploy n8n with the deploy-image chart

We will be using the **deploy-image** chart for deploying a simple application but we will be
using the `helm` client instead of the web interface.

First, we can get the chart values that we can set

```bash
$ helm show values olcf-slate/deploy-image
```

We have a number of values that need to be set to run n8n with the deploy-image chart:

```bash
$ helm install dev olcf-slate/deploy-image \
  --set image=n8nio/n8n \
  --set images.command=/usr/local/lib/node_modules/n8n/bin/n8n \
  --set ingress.enabled=true \
  --set ingress.external=true \
  --set service.port=5678 \
  --set writableVolumes={"/data"} \
  --set ccs.filesystems=olcf-nfs
```

Once it is deployed we can check that the pod is running:

```bash
$ oc get pods
NAME                   READY   STATUS     RESTARTS   AGE
dev-6f56fb698c-jzrql   0/1     Init:0/1   0          2s
```

We can wait for the pod to become ready

```bash
$ oc wait --for=condition=Ready pods --all
pod/dev-6f56fb698c-jzrql condition met
```

And verify that it is ready

```bash
$ oc get pods
NAME                   READY   STATUS    RESTARTS   AGE
dev-6f56fb698c-jzrql   1/1     Running   0          40s
```

Now we can open the URL provided in the `helm install` in a browser and log in build a workflow
with n8n.

## Exercise: Delete the n8n release

Once you are done you can delete all resources associated with the n8n deployment

```bash
$ helm -n stf042 delete dev
release "dev" uninstalled
```

[Next](03_complex_airflow.md)
