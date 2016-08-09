*********************************************
Launching your first instance using Terraform
*********************************************

This section demonstrates how to create your first instance using `Terraform`_.

Terraform is an open source infrastructure configuration and provisioning tool
developed by `Hashicorp`_. Teraform supports the configuration of many kinds of
infrastructure including the Catalyst Cloud. It achieves this with components
known as `providers`_, in the case of the Catalyst Cloud this is the `Openstack
provider`_.

.. _Terraform: https://www.terraform.io/
.. _Hashicorp: https://www.hashicorp.com/
.. _providers: https://www.terraform.io/docs/providers/index.html
.. _Openstack provider: https://www.terraform.io/docs/providers/openstack/index.html

For further information on using Terraform with OpenStack see the following
`video`_ or `blog`_ post.

.. _video: https://www.openstack.org/summit/tokyo-2015/videos/presentation/using-terraform-with-openstack
.. _blog: http://blog.scottlowe.org/2015/11/25/intro-to-terraform/

Install Terraform
=================

Installation of Terraform is very simple, goto the `download`_ page and choose
the zip file that matches your architecture and operating system. Unzip this
file in the location you wish the Terraform binaries to reside on your system.
Terraform is written in `go`_ so there are minimal dependencies.

.. _download: https://www.terraform.io/downloads.html
.. _go: https://golang.org/

.. code-block:: bash

 $ mkdir terraform-first-instance
 $ export TERRAFORM_DIR="$(pwd)/terraform-first-instance"
 $ cd $TERRAFORM_DIR
 $ wget https://releases.hashicorp.com/terraform/0.6.16/terraform_0.6.16_linux_amd64.zip
 $ unzip terraform_0.6.16_linux_amd64.zip

OpenStack credentials
=====================

Before we can run Terraform we need to setup our OpenStack credentials, the
easiest way to achieve this is to make use of environment variables. We will
make use of the standard variables provided by an OpenStack RC file as
described at :ref:`source-rc-file`. These variables are read by the `OpenStack
provider`_ to provide Terraform with permissions to access the Catalyst Cloud
APIs.

.. _OpenStack provider: https://www.terraform.io/docs/providers/openstack/index.html


.. note::

 If you do not source an OpenStack RC file, you will need to set a few
 required authentication arguments in the openstack provider. See the
 Configuration Reference section of the OpenStack provider documentation.

Now that we have a Terraform installation and have setup our OpenStack
credentials we can proceed to build our first instance.

Download the Terraform first instance configuration file
========================================================

It is beyond the scope of this section to explain the syntax of writing
Terraform configuration files, thus we will make use of a predefined example
from the `catalystcloud-orchestration`_ git repository. For more information on
writing Terraform configuration files please consult the Terraform
`documentation`_. The configuration file used in this section can be used as an
example configuration file on which you can build your own templates.

.. _catalystcloud-orchestration: https://github.com/catalyst/catalystcloud-orchestration

.. _documentation: https://www.terraform.io/docs/configuration/index.html

Now let's download the configuration file:

.. code-block:: bash

 $ cd $TERRAFORM_DIR
 $ wget https://raw.githubusercontent.com/catalyst/catalystcloud-orchestration/master/terraform/first-instance/first-instance.tf

Edit this file and change the ``public_key`` under the
``openstack_compute_keypair_v2`` resource. This needs to be contain the actual
public key string, not the path to the public key file. There are a number of
variables in this file that reference particular openstack resources
(``external_gateway``, ``dns_nameservers``, ``image_id`` and ``flavor_id``).
These are configured appropriately for the Porirua region, you will need to
change these if you wish to use the Wellington region.

Run Terraform plan
==================

The terraform plan command will show us the plan that terrafrom intends to
execute:

.. code-block:: bash

 $ ./terraform plan
 Refreshing Terraform state prior to plan...


 The Terraform execution plan has been generated and is shown below.
 Resources are shown in alphabetical order for quick scanning. Green resources
 will be created (or destroyed and then created if an existing resource
 exists), yellow resources are being changed in-place, and red resources
 will be destroyed.

 Note: You didn't specify an "-out" parameter to save this plan, so when
 "apply" is called, Terraform can't guarantee this is what will execute.

 + openstack_compute_floatingip_v2.floatingip_1
     address:     "" => "<computed>"
     fixed_ip:    "" => "<computed>"
     instance_id: "" => "<computed>"
     pool:        "" => "public-net"
     region:      "" => "nz-por-1"

 + openstack_compute_instance_v2.instance_1
     access_ip_v4:               "" => "<computed>"
     access_ip_v6:               "" => "<computed>"
     flavor_id:                  "" => "28153197-6690-4485-9dbc-fc24489b0683"
     flavor_name:                "" => "<computed>"
     floating_ip:                "" => "${openstack_compute_floatingip_v2.floatingip_1.address}"
     image_id:                   "" => "378f3322-740f-4c4d-9864-aebeb41f21ab"
     image_name:                 "" => "<computed>"
     key_pair:                   "" => "first-instance-key"
     metadata.#:                 "" => "1"
     metadata.group:             "" => "test-group"
     name:                       "" => "first-instance"
     network.#:                  "" => "1"
     network.0.access_network:   "" => "0"
     network.0.fixed_ip_v4:      "" => "<computed>"
     network.0.fixed_ip_v6:      "" => "<computed>"
     network.0.floating_ip:      "" => "<computed>"
     network.0.mac:              "" => "<computed>"
     network.0.name:             "" => "private-net"
     network.0.port:             "" => "<computed>"
     network.0.uuid:             "" => "<computed>"
     region:                     "" => "nz-por-1"
     security_groups.#:          "" => "2"
     security_groups.310671339:  "" => "first-instance-sg"
     security_groups.3814588639: "" => "default"
     volume.#:                   "" => "<computed>"

 + openstack_compute_keypair_v2.keypair_1
     name:       "" => "first-instance-key"
     public_key: "" => "ssh-rsa AAAAB3......"
     region:     "" => "nz-por-1"

 + openstack_compute_secgroup_v2.secgroup_1
     description:                  "" => "Network access for our first instance."
     name:                         "" => "first-instance-sg"
     region:                       "" => "nz-por-1"
     rule.#:                       "" => "1"
     rule.836640770.cidr:          "" => "0.0.0.0/0"
     rule.836640770.from_group_id: "" => ""
     rule.836640770.from_port:     "" => "22"
     rule.836640770.id:            "" => "<computed>"
     rule.836640770.ip_protocol:   "" => "tcp"
     rule.836640770.self:          "" => "0"
     rule.836640770.to_port:       "" => "22"

 + openstack_networking_network_v2.network_1
     admin_state_up: "" => "true"
     name:           "" => "private-net"
     region:         "" => "nz-por-1"
     shared:         "" => "<computed>"
     tenant_id:      "" => "<computed>"

 + openstack_networking_router_interface_v2.router_interface_1
     region:    "" => "nz-por-1"
     router_id: "" => "${openstack_networking_router_v2.router_1.id}"
     subnet_id: "" => "${openstack_networking_subnet_v2.subnet_1.id}"

 + openstack_networking_router_v2.router_1
     admin_state_up:   "" => "<computed>"
     distributed:      "" => "<computed>"
     external_gateway: "" => "849ab1e9-7ac5-4618-8801-e6176fbbcf30"
     name:             "" => "border-router"
     region:           "" => "nz-por-1"
     tenant_id:        "" => "<computed>"

 + openstack_networking_subnet_v2.subnet_1
     allocation_pools.#:         "" => "1"
     allocation_pools.0.end:     "" => "10.0.0.200"
     allocation_pools.0.start:   "" => "10.0.0.10"
     cidr:                       "" => "10.0.0.0/24"
     dns_nameservers.#:          "" => "3"
     dns_nameservers.3010225292: "" => "202.78.247.198"
     dns_nameservers.3295368218: "" => "202.78.247.199"
     dns_nameservers.601061661:  "" => "202.78.247.197"
     enable_dhcp:                "" => "1"
     gateway_ip:                 "" => "<computed>"
     ip_version:                 "" => "4"
     name:                       "" => "private-subnet"
     network_id:                 "" => "${openstack_networking_network_v2.network_1.id}"
     region:                     "" => "nz-por-1"
     tenant_id:                  "" => "<computed>"


 Plan: 8 to add, 0 to change, 0 to destroy.

Run Terraform apply
===================

The terraform apply command will execute the plan creating OpenStack resources
for us:

.. code-block:: bash

 $ ./terraform apply
 openstack_compute_keypair_v2.keypair_1: Creating...
   name:       "" => "first-instance-key"
   public_key: "" => "ssh-rsa AAAAB3......"
   region:     "" => "nz-por-1"
 openstack_networking_router_v2.router_1: Creating...
   admin_state_up:   "" => "<computed>"
   distributed:      "" => "<computed>"
   external_gateway: "" => "849ab1e9-7ac5-4618-8801-e6176fbbcf30"
   name:             "" => "border-router"
   region:           "" => "nz-por-1"
   tenant_id:        "" => "<computed>"
 openstack_compute_floatingip_v2.floatingip_1: Creating...
   address:     "" => "<computed>"
   fixed_ip:    "" => "<computed>"
   instance_id: "" => "<computed>"
   pool:        "" => "public-net"
   region:      "" => "nz-por-1"
 openstack_compute_secgroup_v2.secgroup_1: Creating...
   description:                  "" => "Network access for our first instance."
   name:                         "" => "first-instance-sg"
   region:                       "" => "nz-por-1"
   rule.#:                       "" => "1"
   rule.836640770.cidr:          "" => "0.0.0.0/0"
   rule.836640770.from_group_id: "" => ""
   rule.836640770.from_port:     "" => "22"
   rule.836640770.id:            "" => "<computed>"
   rule.836640770.ip_protocol:   "" => "tcp"
   rule.836640770.self:          "" => "0"
   rule.836640770.to_port:       "" => "22"
 openstack_networking_network_v2.network_1: Creating...
   admin_state_up: "" => "true"
   name:           "" => "private-net"
   region:         "" => "nz-por-1"
   shared:         "" => "<computed>"
   tenant_id:      "" => "<computed>"
 openstack_compute_keypair_v2.keypair_1: Creation complete
 openstack_compute_secgroup_v2.secgroup_1: Creation complete
 openstack_compute_floatingip_v2.floatingip_1: Creation complete
 openstack_networking_network_v2.network_1: Creation complete
 openstack_networking_subnet_v2.subnet_1: Creating...
   allocation_pools.#:         "" => "1"
   allocation_pools.0.end:     "" => "10.0.0.200"
   allocation_pools.0.start:   "" => "10.0.0.10"
   cidr:                       "" => "10.0.0.0/24"
   dns_nameservers.#:          "" => "3"
   dns_nameservers.3010225292: "" => "202.78.247.198"
   dns_nameservers.3295368218: "" => "202.78.247.199"
   dns_nameservers.601061661:  "" => "202.78.247.197"
   enable_dhcp:                "" => "1"
   gateway_ip:                 "" => "<computed>"
   ip_version:                 "" => "4"
   name:                       "" => "private-subnet"
   network_id:                 "" => "1913210e-3921-4c9b-b8ab-a097b7c8fc7b"
   region:                     "" => "nz-por-1"
   tenant_id:                  "" => "<computed>"
 openstack_compute_instance_v2.instance_1: Creating...
   access_ip_v4:               "" => "<computed>"
   access_ip_v6:               "" => "<computed>"
   flavor_id:                  "" => "28153197-6690-4485-9dbc-fc24489b0683"
   flavor_name:                "" => "<computed>"
   floating_ip:                "" => "150.242.42.67"
   image_id:                   "" => "378f3322-740f-4c4d-9864-aebeb41f21ab"
   image_name:                 "" => "<computed>"
   key_pair:                   "" => "first-instance-key"
   metadata.#:                 "" => "1"
   metadata.group:             "" => "test-group"
   name:                       "" => "first-instance"
   network.#:                  "" => "1"
   network.0.access_network:   "" => "0"
   network.0.fixed_ip_v4:      "" => "<computed>"
   network.0.fixed_ip_v6:      "" => "<computed>"
   network.0.floating_ip:      "" => "<computed>"
   network.0.mac:              "" => "<computed>"
   network.0.name:             "" => "private-net"
   network.0.port:             "" => "<computed>"
   network.0.uuid:             "" => "<computed>"
   region:                     "" => "nz-por-1"
   security_groups.#:          "" => "2"
   security_groups.310671339:  "" => "first-instance-sg"
   security_groups.3814588639: "" => "default"
   volume.#:                   "" => "<computed>"
 openstack_networking_router_v2.router_1: Creation complete
 openstack_networking_subnet_v2.subnet_1: Creation complete
 openstack_networking_router_interface_v2.router_interface_1: Creating...
   region:    "" => "nz-por-1"
   router_id: "" => "b1a302c2-3369-47bd-ad3f-b85465cd6b72"
   subnet_id: "" => "53dda21d-6e27-43cb-86bf-deb576b10134"
 openstack_compute_instance_v2.instance_1: Still creating... (10s elapsed)
 openstack_networking_router_interface_v2.router_interface_1: Creation complete
 openstack_compute_instance_v2.instance_1: Still creating... (20s elapsed)
 openstack_compute_instance_v2.instance_1: Creation complete

 Apply complete! Resources: 8 added, 0 changed, 0 destroyed.

 The state of your infrastructure has been saved to the path
 below. This state is required to modify and destroy your
 infrastructure, so keep it safe. To inspect the complete state
 use the `terraform show` command.

 State path: terraform.tfstate

Run Terraform delete
====================

The terraform delete command will delete the OpenStack resources we created
previously.

.. note::

 Terraform keeps track of the state of resources using a local file called ``terraform.tfstate``. Terraform consults this file when you destroy resources in order to determine what to delete.

.. code-block:: bash

 $ ./terraform destroy
 Do you really want to destroy?
   Terraform will delete all your managed infrastructure.
   There is no undo. Only 'yes' will be accepted to confirm.

   Enter a value: yes

 openstack_compute_secgroup_v2.secgroup_1: Refreshing state... (ID: 1da4e4a5-5401-4f17-b379-2f397839eb9a)
 openstack_networking_network_v2.network_1: Refreshing state... (ID: 1913210e-3921-4c9b-b8ab-a097b7c8fc7b)
 openstack_compute_floatingip_v2.floatingip_1: Refreshing state... (ID: 580c174a-2972-4597-aedc-f21f5b421e21)
 openstack_networking_router_v2.router_1: Refreshing state... (ID: b1a302c2-3369-47bd-ad3f-b85465cd6b72)
 openstack_compute_keypair_v2.keypair_1: Refreshing state... (ID: first-instance-key)
 openstack_networking_subnet_v2.subnet_1: Refreshing state... (ID: 53dda21d-6e27-43cb-86bf-deb576b10134)
 openstack_compute_instance_v2.instance_1: Refreshing state... (ID: 72776b0d-438e-421d-89fc-3a806eadd3eb)
 openstack_networking_router_interface_v2.router_interface_1: Refreshing state... (ID: 267afa19-f2df-4b17-96da-7a1d09f413b6)
 openstack_networking_router_interface_v2.router_interface_1: Destroying...
 openstack_compute_instance_v2.instance_1: Destroying...
 openstack_compute_instance_v2.instance_1: Still destroying... (10s elapsed)
 openstack_networking_router_interface_v2.router_interface_1: Still destroying... (10s elapsed)
 openstack_networking_router_interface_v2.router_interface_1: Destruction complete
 openstack_networking_subnet_v2.subnet_1: Destroying...
 openstack_networking_router_v2.router_1: Destroying...
 openstack_compute_instance_v2.instance_1: Destruction complete
 openstack_compute_floatingip_v2.floatingip_1: Destroying...
 openstack_compute_keypair_v2.keypair_1: Destroying...
 openstack_compute_secgroup_v2.secgroup_1: Destroying...
 openstack_compute_keypair_v2.keypair_1: Destruction complete
 openstack_compute_floatingip_v2.floatingip_1: Destruction complete
 openstack_networking_subnet_v2.subnet_1: Still destroying... (10s elapsed)
 openstack_networking_router_v2.router_1: Still destroying... (10s elapsed)
 openstack_networking_router_v2.router_1: Destruction complete
 openstack_networking_subnet_v2.subnet_1: Destruction complete
 openstack_networking_network_v2.network_1: Destroying...
 openstack_compute_secgroup_v2.secgroup_1: Still destroying... (10s elapsed)
 openstack_compute_secgroup_v2.secgroup_1: Destruction complete
 openstack_networking_network_v2.network_1: Still destroying... (10s elapsed)
 openstack_networking_network_v2.network_1: Destruction complete

 Apply complete! Resources: 0 added, 0 changed, 8 destroyed.