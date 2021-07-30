# Command Line Interface (CLI)

The OC tool provides CLI access to the OpenShift cluster and it needs to be
installed on your machine. The oc binary offers the same capabilities as the
kubectl binary, but it is further extended to natively support OpenShift
Container Platform features.

* [Download oc for Linux for x86_64](https://downloads-openshift-console.apps.marble.ccs.ornl.gov/amd64/linux/oc.tar)
* [Download oc for Mac for x86_64](https://downloads-openshift-console.apps.marble.ccs.ornl.gov/amd64/mac/oc.zip)
* [Download oc for Windows for x86_64](https://downloads-openshift-console.apps.marble.ccs.ornl.gov/amd64/windows/oc.zip)
* [Download oc for Linux for ARM64](https://downloads-openshift-console.apps.marble.ccs.ornl.gov/arm64/linux/oc.tar)
* [Download oc for Linux for IBM Power, little endian](https://downloads-openshift-console.apps.marble.ccs.ornl.gov/ppc64le/linux/oc.tar)

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
