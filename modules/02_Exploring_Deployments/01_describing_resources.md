# Exploring Resources

In the previous section we focused on the Developer section of the OpenShift web console
but we will be focusing on using the CLI tools.

## Exercise: Create Deployment

**What is a Deployment?**
A Deployment provides declarative updates for Pods and ReplicaSets.
You describe a desired state in a Deployment, and the Deployment Controller changes the actual state to the desired state at a controlled rate. 
You can define Deployments to create new ReplicaSets, or to remove existing Deployments and adopt all their resources with new Deployments.
Reference documentation can be found [here](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).

### 2 Ways to create a Deployment
There are several ways to create a deployment on OpenShift using the CLI tool. 

### Create the Deployment using CLI
The first approach will create a deployment object for us. To create a deployment without YAML, use the following:
```
oc create deployment hello-world-cli --image "gcr.io/google-samples/hello-app:1.0"
```

We can verify this deployment has been created by running:
```
$ oc get deployments
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
hello-world-cli        1/1     1            1           1m
```

### Create the Deployment using YAML
Before starting with the next section, delete the deployment first using:
```
oc delete deployments hello-world-cli
```

Another way is to create the YAML then apply it to the cluster. This is a preferred method when customizing the Deployment is needed 
like adding labels or specifying readiness/liveness probes. We don't have to start YAML from scratch though, native Kubectl provides 
a flag **--dry-run** and **-o yaml** that we can use to generate template to start with:

```
$ oc create deployment hello-world-cli --image "gcr.io/google-samples/hello-app:1.0" --dry-run=client -o yaml | tee > hello-world-cli-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: hello-world-cli
  name: hello-world-cli
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-world-cli
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: hello-world-cli
    spec:
      containers:
      - image: gcr.io/google-samples/hello-app:1.0
        name: hello-app
        resources: {}
status: {}
```

Lets edit this file to look like this:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world-cli
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-world-cli
  template:
    metadata:
      labels:
        app: hello-world-cli
    spec:
      containers:
      - image: gcr.io/google-samples/hello-app:1.0
        name: hello-world-cli
        ports:
        - containerPort: 8080
          protocol: TCP
```

Now we can apply the yaml to the cluster, verification of the deployment can be checked by running **oc get deployments** after running:
```
oc create -f hello-world-cli-deployment.yaml
```

## Exercise: Describing Deployment Components

### Describe Deployment
By creating a Deployment, this by default will create a replicaSet, Pod(s), and the Deployment object. Now that we 
have deployment running in the cluster, we can request information on these different components by using **describe**:
```
$ oc describe deployment hello-world-cli
Name:                   hello-world-cli
...
Selector:               app=hello-world-cli
StrategyType:           RollingUpdate
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=hello-world-cli
  Containers:
   hello-world-cli:
    Image:        gcr.io/google-samples/hello-app:1.0
    Port:         8080/TCP
    Host Port:    0/TCP
  ...
...
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  2m27s  deployment-controller  Scaled up replica set hello-world-cli-d5898598f to 1

```
### Describe ReplicaSet
We don't know the ReplicaSet and Pod names created, as they will be the name of the deployment followed by **-xxx**, but we can  
using the label from the deployment **app: hello-world-cli**:
```
$ oc describe replicasets -l app=hello-world-cli
Name:           hello-world-cli-d5898598f
...
Labels:         pod-template-hash=d5898598f
                app=hello-world-cli
Controlled By:  Deployment/hello-world-cli
Replicas:       1 current / 1 desired
Pods Status:    1 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  pod-template-hash=d5898598f
           app=hello-world-cli
  Containers:
   hello-world-cli:
    Image:        gcr.io/google-samples/hello-app:1.0
    Port:         8080/TCP
    Host Port:    0/TCP
    ...
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  26m   replicaset-controller  Created pod: hello-world-cli-d5898598f-c78rm
```
### Describe Pod
Now lets describe the Pod:
```
$ oc describe pods -l app=hello-world-cli
Name:         hello-world-cli-d5898598f-c78rm
...
Labels:       pod-template-hash=d5898598f
              app=hello-world-cli
...
Status:       Running
...
Containers:
  hello-world-cli:
    Image:          gcr.io/google-samples/hello-app:1.0
    ...
    Limits:
      cpu:     300m
      memory:  500Mi
    Requests:
      cpu:        300m
      memory:     500Mi
...
Events:
  Type    Reason          Age        From                            Message
  ----    ------          ----       ----                            -------
  Normal  Pulling         27m        kubelet, granite4.ccs.ornl.gov  Pulling image "gcr.io/google-samples/hello-app:1.0"
  Normal  Pulled          27m        kubelet, granite4.ccs.ornl.gov  Successfully pulled image "gcr.io/google-samples/hello-app:1.0" in 2.431614684s
  Normal  Started         27m        kubelet, granite4.ccs.ornl.gov  Started container hello-world-cli
```

[Next](02_logs.md)
