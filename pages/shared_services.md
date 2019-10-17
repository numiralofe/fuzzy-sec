# Shared Services
[back](../README.md)

As mentioned on the stack description communication between webservices and processing area is achieved using a messaging system, basically each time that the webservices api receives a request from an end user to process a huge batch of data (file to be processed) or a user browsing the react js page and its action requires some processing, a message is placed on message queue system so that controllers can pull those message's and start processing then.

Another problem is the requirement to control user sessions so that regardless the area /env where the user is being redirected to session information is shared across areas and environments.

To solve the above problems, we have a Shared Services Area that holds the services that will provide solutions for these requirements, since these Shared Services Area seats in the midle between webservices and processing, we will have dedicated vpn's connecting from the cloud provider and the services area so that both areas have access to those services.

For the message queueing the chosen solution is RabbitMQ because its a system that can scale to process and handle thousands of messages in simultaneous.

For the inmemory database, the chosen solution is Redis again by the same reasons as RabbitMQ. 

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

