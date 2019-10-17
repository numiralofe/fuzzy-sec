# Service Discovery / Service Mesh

[back](../README.md)

Consul is a service discovery / mesh solution providing a full featured control plane with service discovery, configuration, and segmentation functionality. Each of these features can be used individually as needed, or they can be used together to build a full service mesh. Consul requires a data plane and supports both a proxy and native integration model.

The key features of Consul are:

- Service Discovery
- Health Checking
- KV Store
- Secure Service Communication
- Multi Datacenter


For our case scenario we will be using consul for:

- Service Discovery: all services regardless if they are running on containers or not will register them selfs in consul, for services that do not have direct integration with consul we can use the nomad service stanza to perform the registration of the service.

```
service {
        name = "myapp"
        port = "http"
        tags = ["myapp version: ${NOMAD_META_SERVICE_VERSION}"]
        check {
            type     = "http"
            port     = "http"
            path     = "/myapp/health"
            interval = "5s"
            timeout  = "2s"
            }
}  
```

- k/v to store all application configurations and deployments: all application configurations and most of the infra configurations related with applications will be store on a git repo and mapped with gonsul directly to consul k/v so that we can easy hot swap them and manage them "as we go".

- enable service mesh - consul connect feature will provide service mesh to some of the areas, sidecar proxy's will be used, as explained [here](../pages/internal_traffic.md) to inter connect services.

- backend storage: Services like vault and nomad need to persist their configurations and consul k/v store will be used as a backend for those services.

