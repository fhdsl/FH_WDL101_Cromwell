

# Using Cromwell at Fred Hutch
Good news! Once you've worked through the Getting Started section, you won't have to do that again! Ongoing use of Cromwell at the Hutch will look a bit more straightforward and we'll discuss the steps to using Cromwell in an ongoing way, the Fred Hutch specific configuration details, and provide some test workflows you can use to test out some of the interfaces we have at the Hutch to Cromwell.  

## Everyday Usage
To get started using Cromwell, you'll first do these steps:

1.  Log into Rhino
2.  Go to your `cromwell-home` directory
3.  Kick off a server job using the command: `./cromwell.sh cromUserConfig.txt`
3.  Wait for a successful response and the node:port information for your server!

That's it!  Now your Cromwell server will run for a week by default (unless you have set a different server length in `cromUserConfig.txt`).  It will be accessible to submit workflows to and execute them whenever you want through multiple mechanisms that we'll describe in the next chapters.  Next week you can simply repeat the above to restart your server and it'll be ready again!  

Don't worry, if you have a workflow that is running at the end of the week and your server job ends, when you start a new server job it will automatically check for the current status of any previously running workflows, then pickup and finish anything that might be left to do.  While you can adjust the configuration of your Cromwell server in your configuration file to run for more than 7 days, we've found that the servers tend to run much faster when they are occasionally "rebooted" like this, and also it is more polite to your lab members to not always have a server running that is not busy coordinating a workflow.  


## Test Workflows
Once you have a server up and running, you'll want to check out our [Test Workflow GitHub repo](https://github.com/FredHutch/wdl-test-workflows) and run through the tests specified in the markdowns there.  The next chapters will guide you through the most common mechanisms for submitting workflows to your server, so you'll want to have cloned this repo to your local computer so you can have the files handy.  They also are useful templates for you to start editing from to craft your first custom workflow later.  

> Note: For those test workflows that use Docker containers, know that the first time you run them, you may notice that jobs aren't being sent very quickly. That is because for our cluster, we need to convert those Docker containers to something that can be run by Singularity. The first time a Docker container is used, it must be converted, but in the future Cromwell will used the cached version of the Docker container and jobs will be submitted more quickly.


## Runtime Variables

Cromwell can help run WDL workflows on a variety of computing resources such as SLURM clusters (like the Fred Hutch cluster), as well as AWS, Google and Azure cloud computing systems.  Using WDL workflows allows users to focus on their workflow contents rather than the intricacies of particular computing platform.  However, there are optimizations of how those workflows run that may be specific to each computing tool or task in your workflow.  Writing your workflow as a WDL allows you to more easily request only the resources each individual task will use each time a job is submitted to the Gizmo cluster.  This allows you to maximize the utilization of the computing resources you request and lets you run workflows much faster than using a single request for a SLURM job and working within that allocation (such as via a `grabnode` process or single bash script).  

We'll discuss some of the available customizations to help you run WDLs on our cluster that still allow those workflows to be portable to other computing platforms.  


### Standard Runtime Variables

These runtime variables can be used on any computing platform, and the values given here are the defaults for our Fred Hutch configuration if there is a default set.  

- `cpu: 1`
  - An integer number of cpus you want for the task. 
- `memory: 2000`
  - An integer number of MB of memory you want to use for the task.  Other formats that are accepted include: `"memory: 2GB"`, `"memory: taskMemory + "GB""` (in this case the memory to use is a variable called `taskMemory` and is specified in a task itself. 
- `docker: `
  - A specific Docker container to use for the task. An example of the value for this variable is: `"ubuntu:latest"`.  No default container is specified in our configuration and this runtime variable should only be used if/when a task should be run inside a docker container, in which case you'll want to specify both the container name and specific version.  If left unset or left out of the runtime block of a task completely, the Fred Hutch configuration will run the task as a regular job and not use docker containers at all.  For the custom Hutch configuration, docker containers can be specified and the necessary conversions (to Singularity) will be performed by Cromwell (not the user).  
  > Note: when docker is used, soft links cannot be used in our filesystem, so workflows using very large datasets may run slightly slower due to the need for Cromwell to copy files rather than link to them.  


### Fred Hutch Custom Runtime Variables
For the `gizmo` cluster, the following custom runtime variables are available (below we show each variable with its current default value). You can change these variables in the `runtime` block for individual tasks in a WDL file. These variables are not variables that will be understood by Cromwell or other WDL engines when the workflow is not being run on the Fred Hutch cluster!  

>Note: when values are specified in the runtime blocks of individual tasks in a workflow, those values will override these defaults for that task only!!


- `walltime: "18:00:00"`
  - A string ("HH:MM:SS") that specifies how much time you want to request for the task. Can also specify >1 day, e.g. "1-12:00:00" is 1 day+12 hours.
- `partition:  "campus-new"`
  - Which cluster partition to use. The default is `campus-new`: other options currently include `restart` or `short` but check [SciWiki](https://sciwiki.fredhutch.org/scicomputing/) for updated information. 
- `modules: ""`
  - A space-separated list of the environment modules you'd like to load (in that order) prior to running the task.  See below for more about software modules.
- `dockerSL: `
  - This is a custom configuration for the Hutch that allows users to use docker and softlinks only to specific locations in Scratch.  It is helpful when working with very large files. An example of the value for this variable is: `"ubuntu:latest"`. Just like the `docker:` runtime variable, only specify this if you want the task to run in a container (otherwise the default will be a non-containerized job). 
- `account: `
  - This allows users who run jobs for multiple PI accounts to specify which account to use for each task, to manage cluster allocations.  An example of the value for this variable is `"paguirigan_a"`, following the  pilastname_f pattern. 

## Managing Software Environments


### Modules
At Fred Hutch we have huge array of pre-curated software modules installed on our SLURM cluster which you can [read about in SciWiki](https://sciwiki.fredhutch.org/scicomputing/compute_scientificSoftware/).   The custom configuration of our Cromwell server allows users to specify one or more modules to use for individual tasks in a workflow.  The desired module(s) can be requested for a task in the `runtime` block of your calls like this:

```
runtime {
  modules: "GATK/4.2.6.1-GCCcore-11.2.0 SAMtools/1.16.1-GCC-11.2.0"
}
```

In this example, we specify two modules, separated by a space (with quotes surrounding them). The GATK module will be loaded first, followed by the SAMtools module.   In this example you'll note the "toolchain" used to build each modules is the same ("GCC-11.2.0").  When you load >1 module for a single task it is important to ensure that they are compatible with each other.  Choose versions built with the same toolchain if you can.  

### Docker
If you want to move your WDL workflow to the cloud in the future, you'll want to leverage Cromwell's ability to run your tasks in Docker containers.  Users can specify docker containers in runtime blocks. Cromwell will maintain a local cache of previously used containers, facilitating the pull of Docker containers and conversion for use.  This behavior allows us to evade rate-limiting by DockerHub and improves speed of your workflows.  We will dig into Docker containers more in the next class.  


Now you're ready to start learning about how to submit our test workflows to your Cromwell server!
