# Integration with batch scheduling

Batch scheduling in this context refers to submitting jobs to the DTN's using our supported [Slurm](https://slurm.schedmd.com/documentation.html) or [LSF](https://www.ibm.com/docs/en/spectrum-lsf/10.1.0?topic=lsf-session-scheduler) scheduler commands.

## Exercise: Submit a job

### Preamble

This exercise will assume that you have a familiarity with creating and running pods on the Marble cluster within Slate. The goal of this exercise is to submit a "Hello World" job onto the DTN's from a Pod running in Marble.

Provided with this documentation is an example pod that is correctly annotated for batch job submission. This Pod is typical in everyway except it has the Batch Scheduling and parallel filesystem annotation

```yaml
ccs.ornl.gov/batchScheduling: "true"
ccs.ornl.gov/fs: olcf
```

The above `ccs.ornl.gov/batchScheduling: "true"` annotation will add the batch commands `bjobs`, `bkill`, `bpeak`, `bsub`, `sbatch` and `squeue` to the `/usr/bin` directory as your Pod is being created.

The `ccs.ornl.gov/fs: olcf` annotation will mount the OLCF Filesystems.

### Steps

The first thing to do is create a Pod that mounts the parallel filesystems and that has Batch Scheduling enabled. This can be done most easily on the command line by using the [CLI for your Operating System](https://console-openshift-console.apps.marble.ccs.ornl.gov/command-line-tools) and the following Pod spec.

If you save the following yaml:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: batch-job-example
  annotations:
    ccs.ornl.gov/batchScheduler: "true"
    ccs.ornl.gov/fs: olcf
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test-jobsubmit
  template:
    metadata:
      labels:
        app: test-jobsubmit
    spec:
      containers:
      - name: test-jobsubmit
        image: "image-registry.openshift-image-registry.svc:5000/openshift/ccs-rhel7-base-amd64:latest"
        args:
        - cat
        stdin: true
        stdinOnce: true
```

into a file named `job-pod.yaml` you would create the Pod with the following command:

```bash
oc apply -f job-pod.yaml
```

Now you should be able to see the pod running:

```bash
oc get pods
```

additionally, you should be able to see that the Pod is mounting the parallel filesystem and our implementations of the above commands:

```bash
oc describe pod batch-job-example
```

If the above conditions are true it is time to submit a job. 

To submit a job, open a terminal into your Pod:

```bash
oc exec -it batch-job-example
```

This should drop you into the / directory of the Pod. From here, change directories into your home area on the filesystem:

```bash
cd /ccs/home/stf002platform_auser
```
* Note your path will be different depending on the group that you are in.

Here you can create a job file:

```bash
#!/bin/env bash

#SBATCH -A stf002platform
#SBATCH -J helloworld
#SBATCH -N 1
#SBATCH -t 1:00:00

echo "Hello world, I am running on node $HOSTNAME"
sleep 10
date
```
* Note, your project name will not be stf002platform unless you are a member of that project.
  
Assuming you saved the job file above into a `hello_world.slurm` file you would now run:

```bash
sbatch -o helloworld_%j.out -e helloworld_%j.err hello_world.slurm
```

to submit the job.

We can now confirm that the job has been submitted with:

```bash
squeue
```

If you see a job named `helloworld` from your user then it worked!

This is just an example of how to specifically use the `sbatch` and `squeue` commands. All of the commands listed above should work as expected from within your pod. 