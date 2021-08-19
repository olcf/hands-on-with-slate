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
oc create deployment hello-world-cli --image "openshift/hello-openshift:latest"
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
$ oc create deployment hello-world-cli --image "openshift/hello-openshift:latest" --dry-run=client -o yaml | tee > hello-world-cli-deployment.yaml
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
        name: hello-world-cli
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
        imagePullPolicy: IfNotPresent
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
Namespace:              stf042
CreationTimestamp:      Thu, 19 Aug 2021 13:41:07 -0400
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=hello-world-cli
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=hello-world-cli
  Containers:
   hello-world-cli:
    Image:        gcr.io/google-samples/hello-app:1.0
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   hello-world-cli-d5898598f (1/1 replicas created)
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
Namespace:      stf042
Selector:       pod-template-hash=d5898598f,app=hello-world-cli
Labels:         pod-template-hash=d5898598f
                app=hello-world-cli
Annotations:    deployment.kubernetes.io/desired-replicas: 1
                deployment.kubernetes.io/max-replicas: 2
                deployment.kubernetes.io/revision: 1
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
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
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
Namespace:    stf042
Priority:     0
Node:         granite4.ccs.ornl.gov/160.91.197.49
Start Time:   Thu, 19 Aug 2021 13:41:07 -0400
Labels:       pod-template-hash=d5898598f
              app=hello-world-cli
Annotations:  k8s.v1.cni.cncf.io/network-status:
                [{
                    "name": "",
                    "interface": "eth0",
                    "ips": [
                        "10.35.31.72"
                    ],
                    "default": true,
                    "dns": {}
                }]
              k8s.v1.cni.cncf.io/networks-status:
                [{
                    "name": "",
                    "interface": "eth0",
                    "ips": [
                        "10.35.31.72"
                    ],
                    "default": true,
                    "dns": {}
                }]
              kubernetes.io/limit-ranger:
                LimitRanger plugin set: cpu, memory request for container hello-world-cli; cpu, memory limit for container hello-world-cli
              openshift.io/scc: ccs-userproject
Status:       Running
IP:           10.35.31.72
IPs:
  IP:           10.35.31.72
Controlled By:  ReplicaSet/hello-world-cli-d5898598f
Containers:
  hello-world-cli:
    Container ID:   cri-o://9a45797ac712b28dfce1703392873bb2a6ea6aeb6f269e6f2fa0d9d7fff7cf88
    Image:          gcr.io/google-samples/hello-app:1.0
    Image ID:       gcr.io/google-samples/hello-app@sha256:60699bc165368192d6c7295b3f837a996a94812d36ef6e7feb2f9c77a558f813
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 19 Aug 2021 13:41:13 -0400
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     300m
      memory:  500Mi
    Requests:
      cpu:        300m
      memory:     500Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-qfgcs (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-qfgcs:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-qfgcs
    Optional:    false
QoS Class:       Guaranteed
Node-Selectors:  region=primary
Tolerations:     ccs.ornl.gov/maintenance:NoSchedule
                 node.kubernetes.io/memory-pressure:NoSchedule
                 node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason          Age        From                            Message
  ----    ------          ----       ----                            -------
  Normal  Scheduled       <unknown>                                  Successfully assigned stf042/hello-world-cli-d5898598f-c78rm to granite4.ccs.ornl.gov
  Normal  AddedInterface  27m        multus                          Add eth0 [10.35.31.72/23]
  Normal  Pulling         27m        kubelet, granite4.ccs.ornl.gov  Pulling image "gcr.io/google-samples/hello-app:1.0"
  Normal  Pulled          27m        kubelet, granite4.ccs.ornl.gov  Successfully pulled image "gcr.io/google-samples/hello-app:1.0" in 2.431614684s
  Normal  Created         27m        kubelet, granite4.ccs.ornl.gov  Created container hello-world-cli
  Normal  Started         27m        kubelet, granite4.ccs.ornl.gov  Started container hello-world-cli
```

[Next](02_logs.md)
