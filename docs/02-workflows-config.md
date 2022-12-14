# FAQ and Reference

## Test Workflows
8.  See our [Test Workflow folder](https://github.com/FredHutch/diy-cromwell-server/tree/main/testWorkflows) once your server is up and run through the tests specified in the markdown there. 
> NOTE: For those test workflows that use Docker containers, know that the first time you run them, you may notice that jobs aren't being sent very quickly.  That is because for our cluster, we need to convert those Docker containers to something that can be run by Singularity.  The first time a Docker container is used, it must be converted, but in the future Cromwell will used the cached version of the Docker container and jobs will be submitted more quickly. 



## Fred Hutch Customization


### Task Defaults and Runtime Variables
For the gizmo backend, the following runtime variables are available that are customized to our configuration.  What is specified below is the current default as written, you can edit these in the config file if you'd like OR you can specify these variables in your `runtime` block in each task to change only the variables you want to change from the default for that particular task.  

- `cpu = 1`
  - An integer number of cpus you want for the task
- `walltime = "18:00:00"`
  - A string of date/time that specifies how many hours/days you want to request for the task
- `memory = 2000`
  - An integer number of MB of memory you want to use for the task
- `partition = "campus-new"`
  - Which partition you want to use, the default is `campus-new` but whatever is in the runtime block of your WDL will overrride this. 
- `modules = ""`
  - A space-separated list of the environment modules you'd like to have loaded (in that order) prior to running the task.  
- `docker = "ubuntu:latest"`
  - A specific Docker container to use for the task.  For the custom Hutch configuration, docker containers can be specified and the necessary conversions (to Singularity) will be performed by Cromwell (not the user).  Note: when docker is used, soft links cannot be used in our filesystem, so workflows using very large datasets may run slightly slower due to the need for Cromwell to copy files rather than link to them.  
- `dockerSL= "ubuntu:latest"`
  - This is a custom configuration for the Hutch that allows users to use docker and softlinks only to specific locations in Scratch.  It is helpful when working with very large files. 


## Guidance and Support
### Monitoring your workflows at Fred Hutch:
I made a shiny app you can use to monitor your own Cromwell server workflows when you have a Cromwell server running on `gizmo` that can be found [here](https://cromwellapp.fredhutch.org/).  If you'd like to roll your own, you can find my shiny app code [here](https://github.com/FredHutch/shiny-cromwell).

### Design Recommendations for WDL workflows at Fred Hutch
See our [SciWiki page](https://sciwiki.fredhutch.org/compdemos/Cromwell/) on Cromwell for more about guidance for how to start structuring and building your workflows as well as how to share them with others on campus in a findable way.  

### R support
I have a basic R package that wraps the Cromwell API allowing you to submit, monitor and kill workflow jobs on `gizmo` from R directly. The package is [fh.wdlR](https://github.com/FredHutch/fh.wdlR).  

### Other Fred Hutch based resources
While additional development is going on to make Cromwell work better in AWS (currently it works well in Google and SLURM among other backends), we are anticipating that it will be more widely available for use with AWS based computing.  To support that there is a growing public data set AWS S3 bucket at `fh-ctr-public-reference-data`.  Contact Amy Paguirigan if you'd like something to be added here and we can help you do that.  

