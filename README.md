# OpenStack Heat template for CaaSP

This repository contains caasp-openstack-heat-templates. This templates build a CaaSP cluster on OpenStack with minimal effort. This templates is suitable for OpenStack users.

It's primary purpose is for building large scale CaaSP clusters on OpenStack for production.

## Requirements

OpenStack CaaSP image requires flavor with minimal resources:

Admin node:

```
  4 vCPU
  16GB RAM
  40GB of disk
```

Master and Worker nodes:

```
  2 vCPU
  8GB RAM
  40GB of disk
```

## Installation procedure

### Preparation

1. Download the latest SUSE CaaS Platform for OpenStack image (for example, SUSE-CaaS-Platform-3.0-OpenStack-Cloud.x86_64-1.0.0-GM.qcow2)
   from https://download.suse.com/Download?buildid=z7ezhywXXRc~

2. Upload the image to Glance:

```
  $ openstack image create --public --disk-format qcow2 --container-format \
    bare --file SUSE-CaaS-Platform-3.0-OpenStack-Cloud.x86_64-1.0.0-GM.qcow2 \
    SUSE-CaaS-Platform-3.0-OpenStack-Cloud
```

3. Install the caasp-openstack-heat-templates package on a machine with SUSE OpenStack Cloud repositories:

```
  $ zypper in caasp-openstack-heat-templates
```

4. The installed templates are located in /usr/share/caasp-openstack-heat-templates.

### INSTALLING TEMPLATES VIA HORIZON

1. In OpenStack Horizon, go to Project › Stacks › Launch Stack.
2. Select File from the Template Source drop-down box and upload the caasp-stack.yaml file.
3. In the Launch Stack dialog, provide the required information (stack name, password, flavor size, external network of your environment, etc.).
4. Click Launch to launch the stack. This creates all required resources for running SUSE CaaS Platform in an OpenStack environment. The stack
   creates one Admin Node, one Master Node, and server worker nodes as specified.

### INSTALL TEMPLATES FROM THE COMMAND LINE

### Steps

1. Specify the appropriate flavor and network settings in the caasp-environment.yaml file.
2. Create a stack in Heat by passing the template, environment file, and parameters:
```
  $ openstack stack create -t caasp-stack.yaml -e caasp-environment.yaml \
    --parameter image=SUSE-CaaS-Platform-3.0-OpenStack-Cloud caasp-stack
```

### INSTALL TEMPLATES FROM COMMAND LINE (FOR MULTIPLE MASTER NODES)

Note: This heat stack with load balancer and multiple master nodes can only be created from command line,
since Horizon does not have support for nested heat templates yet.

### Prerequisites

You need a working load balancer in your Openstack Cloud deployment. There are two load balancer providers
available, 'octavia' and 'haproxy'.

To verify if load balancer with octavia provider or haproxy provider is working correctly in Openstack installation
you can try and create load balancer manually and check if the provisioning_status changes to 'Active'
e.g.

```
neutron lbaas-loadbalancer-create --name test-lb --provider octavia  <SUBNET_ID>
OR
neutron lbaas-loadbalancer-create --name test-lb --provider haproxy  <SUBNET_ID>

neutron lbaas-loadbalancer-show <LOAD_BALANCER_ID>
OR
Openstack CLI:
openstack loadbalancer show <LOAD_BALANCER_ID>
```

Note: Octavia is default load balancer provider in SUSE Openstack Cloud, but it needs a some prerequisites to be setup like, for example
amphora image should be available in Glance. Please refer to documentation in "Installing with
Cloud Lifecyle Manager" guide in Post Installation->Configuring Loadbalancer as a Service->Section 31.3
[Setup of prerequisites](https://www.suse.com/documentation/suse-openstack-cloud-8/book_installation/data/sect1_51_chapter_book_installation.html)
for instructions on how to setup prerequisites which includes steps on "Installing the Amphora Image", "Register the image", and also
steps on "Setup network, subnet, router, security and IP's" to set up a test "lb_net1" network with "lb_subnet1" subnet.

e.g.
```
openstack network create lb_net1
openstack subnet create --name lb_subnet1 lb_net1 --subnet-range 172.29.0.0/24 \
  --gateway 172.29.0.2
openstack router create lb_router1
openstack router add subnet lb_router1 lb_subnet1
openstack router set lb_router1 --external-gateway ext-net
openstack network list

Please use ID of lb_subnet1 as <SUBNET_ID> in neutron lbaas-loadbalancer-create commands above
```

To clean up the environment after the test please delete loadbalancer, router subnet and network
that were created for the test.

```
openstack loadbalancer delete test-lb
openstack subnet delete lb_subnet1
openstack router delete lb_router1
openstack network delete lb_net1
```

### Steps

1. Specify the appropriate flavor and network settings in caasp-multi-master-environment.yaml file.
2. Set master_count to more desired number in caasp-multi-master-environment.yaml file
e.g.

```
master_count: 3
```
Note: The master count has to be set to odd number of nodes  e.g. 1, 3, 5 and so on

3. In a multi master deployment, load balancer is created which points to multiple master nodes for ports
   32000 and 6443. By default the load balancer provider is 'haproxy'.
   You can switch the load balancer provider to 'octavia' by changing the master_lb_provider in caasp-multi-master-environment.yaml file
   e.g.

```
master_lb_provider: haproxy
```
   

4. Create a stack in Heat by passing the template, environment file, and parameters:
```
  $ openstack stack create -t caasp-multi-master-stack.yaml -e caasp-multi-master-environment.yaml \
    --parameter image=SUSE-CaaS-Platform-3.0-OpenStack-Cloud caasp-multi-master-stack
```

5. Floating IP address of the load balancer can be found using the
   following steps

```
openstack loadbalancer list --provider

OR

neutron lbaas-loadbalancer-list
neutron CLI is deprecated and will be removed in the future. Use openstack CLI instead.
+--------------------------------------+------------------------------------+----------------------------------+-------------+---------------------+----------+
| id                                   | name                               | tenant_id                        | vip_address | provisioning_status | provider |
+--------------------------------------+------------------------------------+----------------------------------+-------------+---------------------+----------+
| 0d973d80-1c79-40a4-881b-42d111ee9625 | caasp-stack-master_lb-bhr66gtrx3ue | fd7ffc07400642b1b05dbef647deb4c1 | 172.28.0.6  | ACTIVE              | haproxy  |
+--------------------------------------+------------------------------------+----------------------------------+-------------+---------------------+----------+

Openstack CLI:
openstack loadbalancer show 0d973d80-1c79-40a4-881b-42d111ee9625
OR
neutron lbaas-loadbalancer-show 0d973d80-1c79-40a4-881b-42d111ee9625
neutron CLI is deprecated and will be removed in the future. Use openstack CLI instead.
+---------------------+------------------------------------------------+
| Field               | Value                                          |
+---------------------+------------------------------------------------+
| admin_state_up      | True                                           |
| description         |                                                |
| id                  | 0d973d80-1c79-40a4-881b-42d111ee9625           |
| listeners           | {"id": "c9a34b63-a1c8-4a57-be22-75264769132d"} |
|                     | {"id": "4fa2dae0-126b-4eb0-899f-b2b6f5aab461"} |
| name                | caasp-stack-master_lb-bhr66gtrx3ue             |
| operating_status    | ONLINE                                         |
| pools               | {"id": "8c011309-150c-4252-bb04-6550920e0059"} |
|                     | {"id": "c5f55af7-0a25-4dfa-a088-79e548041929"} |
| provider            | haproxy                                        |
| provisioning_status | ACTIVE                                         |
| tenant_id           | fd7ffc07400642b1b05dbef647deb4c1               |
| vip_address         | 172.28.0.6                                     |
| vip_port_id         | 53ad27ba-1ae0-4cd7-b798-c96b53373e8b           |
| vip_subnet_id       | 87d18a53-ad0c-4d71-b82a-050c229b710a           |
+---------------------+------------------------------------------------+

Openstack CLI:
openstack floating ip list | grep 172.28.0.6
| d636f38b-3197-4b13-ad9a-276072481b0c | fd7ffc07400642b1b05dbef647deb4c1 | 172.28.0.6       | 10.84.65.37         | 53ad27ba-1ae0-4cd7-b798-c96b53373e8b |

Load balancer floating ip address is 10.84.65.37
```

### ACCESSING VELUM SUSE CAAS PLATFORM DASHBOARD

1. After the stack has been created, the Velum SUSE CaaS Platform dashboard runs on the Admin Node. You can access it using the Admin Node's floating IP address.
2. Create an account and follow the steps in the Velum SUSE CaaS Platform dashboard to complete the SUSE CaaS Platform installation.
