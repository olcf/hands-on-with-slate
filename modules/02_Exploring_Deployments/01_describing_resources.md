# Exploring Resources

In the previous section we focused on the Developer section of the OpenShift web console
but we will be focusing on using the CLI tools.

## Exercise: Create Deployment

```
oc create -f deployment.yaml
```

## Exercise: Describe Deployment

```
$ oc get pod,replicaset,deployment
```

```
$ oc describe deployment/hello-world
```

[Next](02_.md)
