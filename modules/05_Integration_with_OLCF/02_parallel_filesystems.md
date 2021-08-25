# Center-wide Parallel Filesystems

Access to the center-wide filesystems is essential for some workloads that run in Slate. Some
examples of these are workflow management and data portal systems.

## Exercise: Mount the Filesystems

First let's deploy a generic busybox shell:

```bash
$ oc apply -f code/deployment.yaml
```

We can exec into a Pod of the Deployment and check that there are no mounts:

```bash
$ oc exec -it deployment/test-mount -- /bin/sh
~ $ ls
bin   dev   etc   home  proc  root  run   sys   tmp   usr   var
~ $
```

Use `Control + D` to exit the shell

Now we can annotate the deployment to add OLCF filesystems

```bash
$ oc annotate deployment/test-mount ccs.ornl.gov/fs=olcf
deployment.apps/test-mount annotated
```

This adds the YAML annotation which triggers a rollout of the Deployment:

```bash
$ oc get deployment/test-mount -o yaml
...
metadata:
  annotations:
    ccs.ornl.gov/fs: olcf
...
```

Now we can exec into the newly deployed Pod of the updated Deployment to find the mounts in
the same place as they would be on a login or compute node of a cluster in NCCS.

```bash
$ oc exec -it deployment/test-mount -- /bin/sh
~ $ ls /
autofs  bin     ccs     dev     etc     gpfs    home    proc    root    run     sw      sys     tmp     usr     var
~ $ ls /ccs/proj/stf042
test-bucket
~ $
```

## Exercise: Clean up

```bash
oc delete -f code/deployment.yaml
```

[Next](03_batch_submission.md)
