# Challenge

## Introduction

Our aim is to draw / show how to build a production environment for a service like HaveIBeenPwnd.

Some constrains:

* It does not have uniform access, with peaks at expected heights (Addition of a new  dataleak) or unexpected (Mentioned in a news story);
* Thousands of millions of data;
* There are workers to process files of new leaks;
* Include monitoring and logging in the design;

## Solution Main Goal

Build an hybrid pipeline solution with availability in multiple regions across multiple providers (SAAS / cloud PAAS / bare metal IAAS) with traffic geo distribution.

## An Hypothetical Fuzz Stack

The stack is composed by several services and bellow you can read a high level resume of this stack and its components:

* **Frontend** is a js application that includes an API client that issues requests to the **Webservices** from the user's browser.

* **Webservices** provides a web API that **Frontend** uses, it needs an SQL database to store persistent data, like user info and configurations and its also responsible by creating job requests that **Controllers** will consume latter.

* **Controller** are a service that manages jobs requested by issued by **Webservices** it's also responsible to create and send tasks to  **Scheduler** so that they are processed by **Workers**.

* **Scheduler** receive tasks from **Controller's** and they are responsible to schedule them on pre reserved time slots on to  **Dispatcher's**.

* **Dispatcher** receive's Tasks, **Scheduler** and are also responsible to send task results back to **Controllers**

* **Workers** are a service that pull requests from **Dispatcher** and process's pool of tasks that **Dispatcher** have.

* **Data Persistency** in the above stack there are 2 main types of data that needs to be persisted, an SQL based one where application configurations and user data is kept, and a non SQL layer that stores full text data gathered and processed by workers and dispatchers.

**To take in account:**
* some services communicate directly with each other.
* some services depend on message brokers or/ in memory to share information between them.

## Main Diagram 

![Main Diagram](images/fuzzsec-Diagram.png?raw=true)

## Resources and Tools

From the infrastructure perspective these would be the tools and resources that would facilitate achieving the main goal:

* [**Providers**: Aws / Google / Packet ](pages/providers.md)
* [**Resources Management**: Terraform / Ansible](pages/resource_management.md)
* [**Service Discovery**: Consul](pages/service_discovery.md)
* [**Applicational Deployments**: Nomad ](pages/applicational_deployment.md)
* [**Applicational Scaling**: Nomad ](pages/applicational_autoscaling.md)
* [**Configuration Management**: Git and Gonsul](pages/configuration_management.md)
* [**Secrets & Credentials Management**: Vault](pages/secrets_and_credentials.md)
* [**External Traffic, Shaping & Geo Distribution**: Geo DNS Policies](pages/external_traffic.md)
* [**Internal Traffic & Service Mesh**: traefik, nginx and consul service mesh.](pages/internal_traffic.md)
* [**Monitoring and Logging**: Elastic Search / Kibana / Grafana / Prometheus.](pages/monitoring_logging.md)

**( Please click on the links from each section above where on a separated page i try to give a more detailed view on how would I use them and some code examples based on my experience.)**