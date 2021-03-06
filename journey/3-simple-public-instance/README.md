- [Objective](#sec-1)
- [Pre requisites](#sec-2)
- [In pratice: OVH cloud: public instance](#sec-3)
- [Going Further](#sec-4)


# Objective<a id="sec-1"></a>

This document is the fourth part of a [step by step guide](../0-simple-terraform/README.md) on how to use the [Hashicorp Terraform](https://terraform.io) tool with [OVH Cloud](https://www.ovh.com/fr/public-cloud/instances/). It will help you create an openstack public instance on the region of your choice.

# Pre requisites<a id="sec-2"></a>

Please refer to the pre requisites paragraph of the [first part](../0-simple-terraform/README.md) of this guide.

# In pratice: OVH cloud: public instance<a id="sec-3"></a>

Here's how to boot a public instance on OVH public cloud using terraform:

Add this snippet in a `.tf` file:

```terraform
# define a remote state backend on swift
terraform {
  backend "swift" {
    container = "demo-public-instance"
  }
}

# configure your openstack provider to target the region of your choice
provider "openstack" {
  region = "${var.region}"
}

# Import Keypair by inlining your ssh public key using terraform interpolation 
# primitives (https://www.terraform.io/docs/configuration/interpolation.html)
resource "openstack_compute_keypair_v2" "keypair" {
  name       = "${var.name}"
  public_key = "${file("~/.ssh/id_rsa.pub")}"
}

# Create your Virtual Machine
resource "openstack_compute_instance_v2" "instance" {
  name        = "${var.name}"

  # Choose your base image from our catalog
  image_name  = "Centos 7"

  # Choose a flavor type
  flavor_name = "s1-8"

  # Target your brand new keypair
  key_pair    = "${openstack_compute_keypair_v2.keypair.name}"

  # Attach your VM to the public network
  network {
    name = "Ext-Net"
    access_network = true
  }
}
```

then run terraform:

```bash
source ~/openrc.sh
terraform init
terraform apply -auto-approve
```

    Initializing the backend...
    
    Successfully configured the backend "swift"! Terraform will automatically
    use this backend unless the backend configuration changes.
    ...
    openstack_compute_keypair_v2.keypair: Creating...
      name:       "" => "demo-public-instance"
      public_key: "" => "ssh-rsa ..."
      region:     "" => "<computed>"
    openstack_compute_keypair_v2.keypair: Creation complete after 0s (ID: demo-public-instance)
    openstack_compute_instance_v2.instance: Creating...
      access_ip_v4:             "" => "<computed>"
      access_ip_v6:             "" => "<computed>"
      all_metadata.%:           "" => "<computed>"
      availability_zone:        "" => "<computed>"
      flavor_id:                "" => "<computed>"
      flavor_name:              "" => "s1-8"
      force_delete:             "" => "false"
      image_id:                 "" => "<computed>"
      image_name:               "" => "Centos 7"
      key_pair:                 "" => "demo-public-instance"
      name:                     "" => "demo-public-instance"
      network.#:                "" => "1"
      network.0.access_network: "" => "true"
      network.0.fixed_ip_v4:    "" => "<computed>"
      network.0.fixed_ip_v6:    "" => "<computed>"
      network.0.floating_ip:    "" => "<computed>"
      network.0.mac:            "" => "<computed>"
      network.0.name:           "" => "Ext-Net"
      network.0.port:           "" => "<computed>"
      network.0.uuid:           "" => "<computed>"
      region:                   "" => "<computed>"
      security_groups.#:        "" => "<computed>"
      stop_before_destroy:      "" => "false"
    openstack_compute_instance_v2.instance: Still creating... (10s elapsed)
    openstack_compute_instance_v2.instance: Still creating... (20s elapsed)
    openstack_compute_instance_v2.instance: Creation complete after 26s (ID: ...)
    
    Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
    
    Outputs:
    
    helper = You can now connect to your instance:
    
     $ ssh centos@a.b.c.d

How fun! You can now ssh into your centos box by pasting the output helper.

Don't forget to destroy your instance once done:

```bash
source ~/openrc.sh
terraform destroy -force
...
```

    openstack_compute_keypair_v2.keypair: Refreshing state... (ID: demo-public-instance)
    openstack_compute_instance_v2.instance: Refreshing state... (ID: da3be2fb-429f-427d-acc3-d5e9262ab460)
    openstack_compute_instance_v2.instance: Destroying... (ID: da3be2fb-429f-427d-acc3-d5e9262ab460)
    openstack_compute_instance_v2.instance: Still destroying... (ID: da3be2fb-429f-427d-acc3-d5e9262ab460, 10s elapsed)
    openstack_compute_instance_v2.instance: Destruction complete after 10s
    openstack_compute_keypair_v2.keypair: Destroying... (ID: demo-public-instance)
    openstack_compute_keypair_v2.keypair: Destruction complete after 0s
    
    Destroy complete! Resources: 2 destroyed.

# Going Further<a id="sec-4"></a>

Next time we'll introduce more advanced features of our cloud, such as ports and security groups.

See you on [the fifth step](../4-advanced-public-instances/README.md) of our journey.
