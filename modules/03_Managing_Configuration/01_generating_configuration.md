# Generating Configuration

The YAML (or JSON) that is used to describe objects in Kubernetes is incredibly descriptive but it
can be intimidating to new users. The object API is backed by a well-defined schema which can be
found in the [Kubernetes documentation](https://kubernetes.io/docs/reference/kubernetes-api/) but
it is difficult to parse what is required and what is optional.

The `oc` and `kubectl` command line tools have the ability to generate boilerplate configuration which
can make it faster to get started when developing configuration from scratch.

## Generating Application Deployment YAML

Let's generate the configuration needed to deploy a standard application stack: a **Deployment** object to manage the pods and
actually run the container image and **Service** and **Route** objects to handle getting the network traffic to the pods.

```bash
$ oc create deployment test --image=busybox --dry-run=client -o yaml | tee > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: test
  name: test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: test
    spec:
      containers:
      - image: busybox
        name: busybox
        resources: {}
status: {}
```

```bash
$ oc create service clusterip test --tcp=8080:8080 --dry-run=client -o yaml | tee > service.yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: test
  name: test
spec:
  ports:
  - name: 8080-8080
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: test
  type: ClusterIP
status:
  loadBalancer: {}
```

```bash
$ oc create route edge --service=test --port=8080 --dry-run=client -o yaml | tee > route.yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  creationTimestamp: null
  name: test
spec:
  port:
    targetPort: 8080
  tls:
    termination: edge
  to:
    kind: ""
    name: test
    weight: null
status: {}
```

```bash
$ oc create -f .
deployment.apps/test created
route.route.openshift.io/test created
service/test created
```
