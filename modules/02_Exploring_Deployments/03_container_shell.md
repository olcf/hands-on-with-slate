# Shell in Running Container

Sometimes you need to access the container in the cluster to debug, this can be done using the **exec** function:

## Exercise: Get a Shell

```
$ oc exec -it deploy/hello-world-cli -- /bin/sh
~ $
~ $ exit
```

[Next](04_resources.md)