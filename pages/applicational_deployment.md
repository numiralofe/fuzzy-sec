# Applicational Deployment 
[back](../README.md)


## What is Nomad?

Nomad is a tool that makes part of the Hashicorp Stack for managing a cluster of machines and running applications on them. Nomad abstracts away machines and the location of applications, and instead enables users to declare what they want to run and Nomad handles where they should run and how to run them.

The key features of Nomad are:

Docker Support: Nomad supports Docker as a first-class workload type. Jobs submitted to Nomad can use the docker driver to easily deploy containerized applications to a cluster. Nomad enforces the user-specified constraints, ensuring the application only runs in the correct region, datacenter, and host environment. Jobs can specify the number of instances needed and Nomad will handle placement and recover from failures automatically.

Operationally Simple: Nomad ships as a single binary, both for clients and servers, and requires no external services for coordination or storage. Nomad combines features of both resource managers and schedulers into a single system. Nomad builds on the strength of Serf and Consul, distributed management tools by HashiCorp.

Multi-Datacenter and Multi-Region Aware: Nomad models infrastructure as groups of datacenter which form a larger region. Scheduling operates at the region level allowing for cross-datacenter scheduling. Multiple regions federate together allowing jobs to be registered globally.

Flexible Workloads: Nomad has extensible support for task drivers, allowing it to run containerized, virtualized, and standalone applications. Users can easily start Docker containers, VMs, or application runtimes like Java. Nomad supports Linux, Windows, BSD and OSX, providing the flexibility to run any workload.

Built for Scale: Nomad was designed from the ground up to support global scale infrastructure. Nomad is distributed and highly available, using both leader election and state replication to provide availability in the face of failures. Nomad is optimistically concurrent, enabling all servers to participate in scheduling decisions which increases the total throughput and reduces latency to support demanding workloads. Nomad has been proven to scale to cluster sizes that exceed 10k nodes in real-world production environments.

## High Level Overview

Within each example dc's (eu-dc / us-dc1), we have both clients and servers, servers are responsible for accepting jobs from users, managing clients, and computing task placements. Each region may have clients from multiple datacenter, allowing a small number of servers to handle very large clusters.

Regions are fully independent from each other, and do not share jobs, clients, or state, they are loosely-coupled using a gossip protocol, which allows users to submit jobs to any region or query the state of any region transparently. Requests are forwarded to the appropriate server to be processed and the results returned. Data is not replicated between regions.

## Nomad Cluster Definition
Because Nomad operates at a regional level, federation is part of Nomad core. Federation enables users to submit jobs or interact with the HTTP API targeting any region, from any server, even if that server resides in a different region, each cluster can define its own meta tag info in order to group clients and that can be used by those on allocations and jobs.

![Main Diagram](../images/nomad_architecture.png?raw=true)

## Architecture Implementation

We have one or multiple servers per areas running docker.io, consul agent and nomad agent, within the current consul cluster we will have installed and running the nomad cluster, meaning that from the server side, we have a consul / nomad cluster running server version of both applications, on the client side we have multiple machines distributed per area , each area is segregated and independent, meaning that traffic between dc's is restricted.
Depending on the job specification as soon as it is submitted, the nomad cluster will handle where the app should run and how to run it, further on I will document an example job.

![Main Diagram](../images/nomad_deployments.png?raw=true)

## blue / green and /or canary Deployments
Imagine a hypothetical API service which has 2 instances deployed to production with version 1.3, and we want to safely upgrade to version 1.4. We want to create 2 new instances at version 1.4 and in the case that they are operating correctly we want to promote them and take down the 2 versions running 1.3. In the event of failure, we can quickly rollback to 1.3.When we change the job to run the "api-server:1.4" image, Nomad will create 2 new allocations without touching the original "api-server:1.3" allocations. 

````
  group "workers" {
    count = 5  // set the number of workers to 5 intances

    update {
      max_parallel     = 1
      canary           = 2 // set 2 canary app intances on release
      min_healthy_time = "30s"
      healthy_deadline = "10m"
      auto_revert      = true
      auto_promote     = false
    }
  }
  ````
  
## raw / binary payloads

I would like to make a special mention to the raw driver support by nomad, this driver is used to execute static linked binaries directly on cluster hosts. On the main diagram the areas marked with "gears" means that raw payload is supported meaning that, lets imagine an hypothetical artifactory where we have binaries with our applications stored we can instruct using the nomad job to fetch those binaries from the artifavtory directly and execute them on the remote nodes without the need of the docker overhead.


Example:
```
» Task Configuration
task "worker" {
  driver = "raw_exec"

    config {
        command = "my_service --arg1=test1 --arg2=test2"
    }

  artifact {
    source = "https://artifactory.server/my_service"

    options {
      checksum = "md5:123445555555555"
    }
  }

}  
```

## Web hook deployment example

As an example lets imagine the following scenario for example hcl stanza where we are "hard coding" the application version to be deployed:

````
task "webservice" {
  driver = "docker"

  config {
    image = "webserver:3.2"
    labels {
      group = "webserver"
    }
  }
}
````

If instead we instruct the hcl stanza to fetch the key value for the image version from a value defined in consul k/v store, like this:

````
task "webservice" {
  driver = "docker"

  config {
    image = "webserver:{{ deployments/webserver_version }}"
    labels {
      group = "webserver"
    }
  }
}
````

From jenkins / drone we just need to create/enable a simple web hook that updates a git repo file that by its turn will be automatically pushed to consul k/v by gonsul, this action will automatically trigger nomad to execute the deployment/update for the service in question with the new version.




