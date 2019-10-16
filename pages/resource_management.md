# Resources Management
[back](../README.md)

All resources, regardless of the provider will be managed by terraform and we will occasionally use ansible to setup and configure some resources in those situations where it's not possible/easier to do it with terraform.

The list of resources to be managed by Terraform  will be:

- All Virtual Machines (disks and specs)
- Image to be used on deployment of VMs
- Networks /VPCs
- Routes
- Firewall Rules
- Tags
- DNS Records
- External IPs
- VPNs
- Managed Services
- Cloud Load Balancers

The code available on the terraform repos will represent the state of the infrastructure so that we can tell what’s currently deployed and how it’s configured, resources will be managed on a per project base, being that will be split per branch and provider but using common modules.

A vars folder will be created per provider and we try to reuse components as much as possible between providers by making full extent of terraform modules features.

* main.tf - call modules, locals and data-sources to create all resources
* variables.tf - contains declarations of variables used in main.tf
* outputs.tf - contains outputs from the resources created in main.tf


You can find on the link bellow and full code example written by me 2 years ago:

        https://github.com/numiralofe/automation/tree/master/commitApp_POC/terraform

Terraform pre builded Modules to deploy multiple services:

        https://github.com/hashicorp/terraform-aws-nomad
        
        https://github.com/hashicorp/terraform-azurerm-nomad



