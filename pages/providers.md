# Providers
[back](../README.md)


Taking in consideration the level of abstraction that the proposed solution has, I am assuming that this stack can be deployed in any of the main [providers](https://www.terraform.io/docs/providers/index.html) that terraform supports, all the [providers](https://www.terraform.io/docs/providers/index.html) have their weakness and strengths and  depending on the geographical area where we are aiming to operate and the type of load that its going to be executed, etc, it can make sense to use one or the other. 

As an example, the Chinese market as its own particularities due to the existing restrictions, china fw and legal restrictions, pricing is also important since resources outside of "common areas" (EU/US) tend to be much more expensive, also some software components used to manage resources and automation have to be carefully chosen, for instance, for k8s managed service each provider has its own implementation (EKS / GKS) and they have different details in each implementation resulting in a double effort to maintain different configurations, another good example is data-sources, mostly its not possible to setup a managed Postgres/Mysql cluster expanded/available between providers, as a result of all the mentioned points and others not mentioned, key strategy decisions must be taken upfront in order to build the delivery pipeline as flexible as possible.

## Providers in Scope:

• AWS - https://aws.amazon.com/

• Google - https://cloud.google.com

• Azure - https://azure.microsoft.com

• Packet - https://www.packet.com/

• LeaseWeb - https://www.leaseweb.com/dedicated-servers


Here apart from the more traditional cloud providers AWS/Google/Azure i am also adding packet and LeaseWeb, would like to make a special mention to them since:

Packet provide a full automated api fully integrated with Terraform allowing companies to use "bare metal as if they were EC2/GCloud Instances" but there are no shared resources or required hypervisors to gum up the works and impact workload, according to them, by delivering only "hardware as a service", they are able to provide vastly more performance infrastructure at up to 50-60% less cost

**disclaimer:** I don't have any type of affiliation with packet :) I am just making a special mention to them because i find their overall concept real interesting, pls give it 5mins read on their website if you don't know them yet.

LeaseWeb has perhaps one of the most competitive prices in the European market, they would be ideal perhaps to run large batch's of jobs that temporary require massive cpu usage. 


