# Build an Image using OpenShift

We will use the integrated OpenShift container build service as well as the integrated
container registry to build and push a container image in OpenShift.

## Exercise: Create an ImageStream and BuildConfig

We will be using the OpenShift console client tool for this section and while the OpenShift
build service has a lot of functionality, we will be focusing on a simple Dockerfile workflow
in a local directory.

We will be creating two objects: ImageStream and BuildConfig

The ImageStream represents a container image that is stored in the OpenShift integrated container
registry. Images can be pushed into and pulled from the registry both inside and outside of the
cluster.

A BuildConfig describes how the integrated contianer build service actually builds the container.
In this example, our configuration is minimal but the BuildConfig has more advanced configuration
options such as automatically build containers from configuration stored in a git repo.

Using the `oc` client utility we can create a "binary" input build which means that the Build input
will come from our local machine.

```
$ oc new-build --name example-build --binary
    * A Docker build using binary input will be created
      * The resulting image will be pushed to image stream tag "example-build:latest"
      * A binary build was created, use 'oc start-build --from-dir' to trigger a new build

--> Creating resources with label build=example-build ...
    imagestream.image.openshift.io "example-build" created
    buildconfig.build.openshift.io "example-build" created
--> Success
```

We can kick off the build by specifying the directory to upload as the Build context (this directory
can be found inside this module in the git repository):

```
$ tree image/
image/
├── Dockerfile
├── index.html
└── nginx.conf
```

> Note: We make a few changes in the nginx.conf file including sending logs to stdout and stderr,
turning off pid file creation, and running as a specific user id. The logging is sent to stdout
because OpenShift will store all logs from the container. We turn off pidfile creation because
it is not necessary in our enviornment. We do not specify a user id to run as because all container
processes in Slate run as the automation user for the project.

```
$ oc start-build example-build --from-dir=image --follow
```

## Exercise: Update Deployment with New Image

Follow the same procedure from before and update our Helm chart deployment with a new image.

- Image: **example-build:latest**

[Next](06_storage.md)
