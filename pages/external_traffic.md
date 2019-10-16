# External Traffic
[back](../README.md)

I would solve the geolocation routing by using AWS Route 53 DNS Service (or another similar approach) since this type of solution allows to choose the resources that serve traffic based on the geographic location of the users by using the location that DNS queries originate from and with that we can route all requests from Europe to be routed to load balancer's in europe and/or requests from USA to us.

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

SSL termination and offload will be done on the top root load balancers.




