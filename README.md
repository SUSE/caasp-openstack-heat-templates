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

1. Download the latest SUSE CaaS Platform for OpenStack image (for example, SUSE-CaaS-Platform-2.0-OpenStack-Cloud.x86_64-1.0.0-GM.qcow2) from https://download.suse.com.
2. Upload the image to Glance:

```
  $ openstack image create --public --disk-format qcow2 --container-format \
    bare --file SUSE-CaaS-Platform-2.0-for-OpenStack-Cloud.x86_64-2.0.0-GM.qcow2 \
    CaaSP-2
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
4. Click Launch to launch the stack. This creates all required resources for running SUSE CaaS Platform in an OpenStack environment. The stack creates one Admin Node, one Master Node, and server worker nodes as specified.

### INSTALL TEMPLATES FROM THE COMMAND LINE

1. Specify the appropriate flavor and network settings in the caasp-environment.yaml file.
2. Create a stack in Heat by passing the template, environment file, and parameters:
```
  $ openstack stack create -t caasp-stack.yaml -e caasp-environment.yaml \
    --parameter image=CaaSP-2 caasp-stack
```

### ACCESSING VELUM SUSE CAAS PLATFORM DASHBOARD

1. After the stack has been created, the Velum SUSE CaaS Platform dashboard runs on the Admin Node. You can access it using the Admin Node's floating IP address.
2. Create an account and follow the steps in the Velum SUSE CaaS Platform dashboard to complete the SUSE CaaS Platform installation.
