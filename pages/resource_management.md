# Resources Management
[back](../README.md)

**Terraform & Environment Bootstrap**
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



**Packer & Image Templating**

Due to the way that the auto-scaling process works at cloud providers (aws/gcloud/azure) and also because its important to keep consistency across datacenter resources we should ideally use the same set of software across all of them, also for some situations (namely auto-scaling actions) its more efficient to have pre installed software components that we need so that they are available on boot time and avoid the extra latency that download and install them brings. 

To solve the mentioned issues we need to have pre baked base OS images, for that i am including  a process that manages and builds images using packer (Hashicorp Packer).

* common packer configuration file that builds and targets aws / gcloud / azure / kvm.
* env variables that defines which version's and software we want to bake in.
* have a drone/jenkins file that picks up changes at the packer main config file and automatically builds the OS images and push them to the cloud providers image artifactory.

**Bare Metal**

On this point we pretty much depend on the provider's that we decide to use, i would prefer providers like Packet or LeaseWeb (mentioned on the providers page) since those have available API's that we can use to interact and create physical resources with pre installed OS Versions allowing us this way to control and automate installation of base components.

We can off course install OpenStack, kvm Proxmox or any other complex solution, but trying to follow the ***KISS*** principle and if workload types are common and typified i would like to make a special mention to the following combo :)

***kvm + nomad + consul + packer***

Assuming that we have the process described above that allows us to create golden-images that we use on the cloud providers, we could reuse the same process to build quemu images using packer.

* https://github.com/tylert/packer-build

And then:

1 - request a new physical machine by using the provider api and install nomad and consul agents.

An example using packet:

```
curl -v -X POST -H 'X-Auth-Token: <API-KEY>' -H 'Content-Type: application/json' -d '
{
 "facility": "ewr1",
 "plan": "t1.small.x86",
 "hostname": "string",
 "description": "string",
 "billing_cycle": "hourly",
 "operating_system": "debian_10",
 "userdata": "
 #!/bin/bash 
export DEBIAN_FRONTEND=noninteractive 
apt-get update && apt-get upgrade -y 
apt-get install unzip -y
wget -q https://releases.hashicorp.com/consul/"$CONSUL_VERSION"/consul_"$CONSUL_VERSION"_linux_amd64.zip -O /tmp/consul.zip
mkdir -p /opt/consul/
mkdir -p /opt/consul/consul.d
unzip -oq /tmp/consul.zip -d /opt/consul/
chmod 0750 /opt/consul/consul
chown root:root /opt/consul/consul
ln -s /opt/consul/consul /usr/bin/consul 
rm /tmp/consul.zip
wget -q https://releases.hashicorp.com/nomad/"$NOMAD_VERSION"/nomad_"$NOMAD_VERSION"_linux_amd64.zip -O /tmp/nomad.zip
mkdir -p /opt/nomad/nomad.d 
mkdir -p /opt/nomad/credentials
unzip -oq /tmp/nomad.zip -d /opt/nomad/
chmod 0750 /opt/nomad/nomad
chown root:root /opt/nomad/nomad
ln -s /opt/nomad/nomad /usr/bin/nomad
rm /tmp/nomad.zip
 ",
 "locked": "false"
}' "https://api.packet.net/projects/<Project-ID>/devices"
```

2. Boot nomad and consul and connect them to the existing cluster

***consul bootstrap:***
```
consul agent -retry-join=["eu-dc1-fuzz-sec"]
```

***nomad bootstrap:***
```
nomad server join eu-dc1-fuzz-sec:4648
```

3 -  And then because we have templated quemu images we can use nomad to start deploying quemu workloads to the created resource.

nomad job to deploy and start vm's on the new resource:

```
task "workload" {
  driver = "qemu"

  config {
    image_path  = "local/golden-image.img"
    accelerator = "kvm"
    args        = ["-nodefaults", "-nodefconfig"]
  }

  # Specifying an artifact is required with the "qemu"
  # driver. This is the # mechanism to ship the image to be run.
  artifact {
    source = "https://myartifactory.fuzzy-sec.com/images/golden-image.img"

    options {
      checksum = "md5:123445555555555"
    }
  }
```

**Considerations:** This example is on how to boot Virtual Machines but the same approach can be used to install docker and run only docker container workloads, or even raw_executions workloads.




