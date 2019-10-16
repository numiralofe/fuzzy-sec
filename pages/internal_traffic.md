# Internal Traffic
[back](../README.md)

## Service Mesh / Sidecars

Consul Connect provides service-to-service connection authorization and encryption using TLS, like other service mesh softwares it comes with a proxy that’s deployed as a sidecar and the proxy transparently secures communication between micro services and enables policy definition through intentions, so that we can define what can communicate with and not, for our use case inside each area services will communicates between them using connect, as an example on the processing area, worker pools and dispatcher pools will communicate between them using connect, the same goes for dispatcher's and schedulers.

We can declare the service on consul using nomad hcl stanza specification:

nomad stanza that deploys the worker instances:
```
            config {
                command = "/usr/local/bin/consul"
                args    = [
                    "connect", "proxy",
                    "-service", "worker",
                    "-service-addr", "${NOMAD_ADDR_worker_tcp}",
                    "-listen", ":${NOMAD_PORT_tcp}",
                    "-register",
                ]
            }
```


Then we can reuse it on the dispatcher jobs and make the workers service available inside the dispatcher containers as a local port, avoiding this way internal load balancing between running containers of the services:

```
        task "proxy" {
            driver = "raw_exec"

            config {
                command = "/usr/local/bin/consul"
                args    = [
                    "connect", "proxy",
                    "-service", "dispatcher",
                    "-upstream", "worker:${NOMAD_PORT_tcp}",
                ]
            }
```

By using consul connect we:

* secure connections between endpoints by using TLS.
* get grained control from what connects with what
* avoid the overhead of load balancers to manage traffic between services since they will communicate directly.

