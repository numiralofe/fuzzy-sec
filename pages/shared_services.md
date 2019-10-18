# Shared Services
[back](../README.md)

As mentioned on the stack description communication between webservices and processing area is achieved using a messaging system, basically each time that the webservices api receives a request from an end user to process a huge batch of data (file to be processed) or a user browsing the react js page and its action requires some processing, a message is placed on message queue system so that controllers can pull those message's and start processing then.

Another problem is the requirement to control user sessions so that regardless the area /env where the end user is coming from the session information is stored on a system that can be shared across areas and environments.

To help solving the mentioned problems, we have a Shared Services Area that holds the services that will provide solutions for those requirements, since Shared Services Area seats in the middle between webservices and processing, we will have dedicated VPN's connecting the VPC's from the cloud provider's so that both webservices and/or processing area's have access to this shared services.

For the message queueing the chosen solution is RabbitMQ because its a system that can scale to process and handle thousands of messages in simultaneous.

For the in memory database, the chosen solution is Redis because:
- we are just using to store and persist user session data.
- its by desing easy to install and maintain

We need to take in account that any of this data is meant to be persisted.

On this area we also have a nomad cluster and both services are deployed inside the cluster, since this will allow us to set scaling policies for both services in case its required to scale them.


```
job "rabbit" {

  datacenters = ["eu-sharedservices"]
  type = "service"

  group "cluster" {
    count = 3


    task "rabbit" {
      driver = "docker"

      config {
        image = "pondidum/rabbitmq:consul"
        hostname = "${attr.unique.hostname}"
        port_map {
          amqp = 5672
          ui = 15672
          epmd = 4369
          clustering = 25672
        }
      }

      env {
        RABBITMQ_ERLANG_COOKIE = "generate_a_guid_-_or_something_for_this"
        RABBITMQ_DEFAULT_USER = "test"
        RABBITMQ_DEFAULT_PASS = "test"
        CONSUL_HOST = "${attr.unique.network.ip-address}"
      }

      resources {
        network {
          port "amqp" { static = 5672 }
          port "ui" { static = 15672 }
          port "epmd" { static = 4369 }
          port "clustering" { static = 25672 }
        }
      }

      service {
        name = "rabbitmq"
        port = "ui"
        check {
          name     = "alive"
          type     = "tcp"
          interval = "10s"
          timeout  = "2s"
        }
      }

    }
  }
}
```

