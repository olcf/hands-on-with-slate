# Command Line Interface (CLI)

The OC tool provides CLI access to the OpenShift cluster. It needs to be
installed on your machine.

## Exercise: Logging in with the CLI

|Cluster|URL|
|---|---|
|Marble (Moderate Production cluster with access to Summit/Alpine)|`https://api.marble.ccs.ornl.gov`|
|Onyx (Open Production Cluster with access to Wolf)|`https://api.onyx.ccs.ornl.gov`|


```bash
$ oc login <URL>
Authentication required for <URL> (openshift)
Username: user
Password:
Login successful.
```

```bash
$ oc whoami
user
```

[Next](02_web_console.md)
