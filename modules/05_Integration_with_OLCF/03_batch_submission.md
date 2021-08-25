# Cross Job Submission to Other Clusters

Batch scheduling in this context refers to submitting jobs to the DTN's using our supported
[Slurm](https://slurm.schedmd.com/documentation.html) or [LSF](https://www.ibm.com/docs/en/spectrum-lsf/10.1.0?topic=lsf-session-scheduler)
scheduler commands.

## Exercise: Submit a job

The goal of this exercise is to submit a "Hello World" job onto the DTN's from a Pod
running in Marble.

Provided with this documentation is an example pod that is correctly annotated for
batch job submission. This Pod is typical in everyway except it has the Batch Scheduling
and parallel filesystem annotation

```yaml
metadata:
  annotations:
    ccs.ornl.gov/batchScheduling: "true"
    ccs.ornl.gov/fs: olcf
```

The **batchScheduling** annotation instructs the system to inject a number of the common batch
system commands for both LSF and Slurm directly into the running container. The only requirement
in the container is to have the `ssh` binary.

First step is to create the Deployment:

```bash
oc apply -f code/job-submit.yaml
```

To test job submission, we will open a shell in the running Deployment:

```bash
oc exec -it batch-job-example
```

This should drop you into the / directory of the Pod. From here, change directories into your home area on the filesystem:

```bash
cd $HOME
```

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

> Note: your project name will not be stf002platform unless you are a member of that project.
  
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
