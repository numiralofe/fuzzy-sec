# External Traffic
[back](../README.md)

For the frontend application, that is a js application, in order to scale we would deploy it on any CDN service, for this we could use google/azure/aws buckets or a dedicated service like cloudfare, by doing this we would delegate to those providers the scaling problem of serving the frontend js app at scale.

Then at the webservice layer i would solve the geolocation routing by using AWS Route 53 DNS Service (or another similar approach) since this type of solution allows to choose the resources that serve traffic based on the geographic location of the users by using the location that DNS queries originate from and with that we can route all requests from Europe to be routed to load balancer's in europe and/or requests from USA to us.

If needed we can go even further and divide EU area in small multiple ones and then have redundancy based on location/multiple data center, for instance we could route all EU traffic to a main domain / endpoint and then have multiple sub-domains that maps each region and traffic could be split accordingly.

Example.
````
+-- fuzz-sec.com
|   |
|   +-- eu.fuzz-sec.com
|   |	|-- region-eu-01.fuzz-sec.com
|   |   |-- region-eu-02.fuzz-sec.com
|   +-- us.fuzz-sec.com
|   |	|-- region-us-01.fuzz-sec.com
|   |   |-- region-us-02.fuzz-sec.com
````

On the example above, fuzz-sec.com would contain the A Records from eu.fuzz-sec.com and us.fuzz-sec.com so that end users requests on each continent would be redirected to their closest location, once there, we could have a main load balancer that receives all requests for Europe region (or US depending on the origin request) and then map inside the region to one of the active locations (eu-dc1 / eu-dc2 or us-dc1 / us-dc2), with this scenario we achieve traffic geo distribution and at the same time full stack redundancy since in case one of the regions fails we have a second pool inside the same geographical area but on a different data center.

## SSL Offload

For the webservices API public endpoint SSL termination and offload will be done on the top root load balancers of each area/provider.


## WebServices Public Endpoint

We would use traefik to manage public endpoints for webservices that expose a public api, with traefik we would use the following features to solve multi domain and multi env problem:


* Since it has direct integration with consul we can read services from there and declare  consul dynamic properties so that apps are configured according to the place where they are running.
* by using tags we can easily handle multi domain pools configurations and we can set them at the hcl level.

```
service {
        name = "${NOMAD_META_SERVICE_ALIAS}-${meta.environment}"
        port = "http"
        tags = ["webservices version: ${NOMAD_META_SERVICE_VERSION}",
        "traefik.enable=true",
        "traefik.frontend.entryPoints=https",
        "traefikdmz.frontend.rule=Host:eu-dc1.fuzzy-sec.com" ]
}   
```

