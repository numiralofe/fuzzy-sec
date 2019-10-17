# Internal Traffic
[back](../README.md)

## Service Mesh / Sidecars

Consul Connect provides service-to-service connection authorization and encryption using TLS, like other service mesh softwares it comes with a proxy that’s deployed as a sidecar and the proxy transparently secures communication between micro services and enables policy definition through intentions, so that we can define what can communicate with and not, for our use case inside each area services will communicates between them using connect, as an example on the processing area, worker pools and dispatcher pools will communicate between them using connect, the same goes for dispatcher's and schedulers.

We can declare the service on consul using nomad hcl stanza specification:

nomad stanza that deploys the worker instances:
```
service {
    name = "“controllers"
    port = "8000"

    connect {
        sidecar_service {}
        }
}
```

Then we can reuse it on the workers jobs and make the controllers service available inside the workers containers as a local port, avoiding this way internal load balancing between running containers of the services:

```
service {
       name = workers
       port = “http_port”

       connect {
         sidecar_service {
           proxy {
             upstreams {
               destination_name = “controllers"
               local_bind_port = 8000
             }
           }
         }
       }
   }   
```

![Service Mesh](../images/fuzzsec-ServiceMesh.png?raw=true)

By using consul connect we:

* secure connections between endpoints by using TLS.
* get grained control from what connects with what
* avoid the overhead of load balancers to manage traffic between services since they will communicate directly.
