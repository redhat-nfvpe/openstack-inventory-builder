# openstack-inventory-builder

A set of simple playbooks that can be called from
[AWX](https://github.com/ansible/awx) to instantiate a set of virtual machines
on an OpenStack cluster.

## Usage

You can use this repository as the source for a project, then create a _Job
Template_ and pass information into the _Extra Variables_ section to instantiate
one or more virtual machines in your OpenStack environment.

### Create a Project

In AWX (Tower), create a Project and use the SCM type of `Git`. Use this
repository as the source for that project. Save the project and move to the
Inventory section.

### Create a Credential

You need to create a Credential so that we can authenticate and speak with the
cloud API. You're going to need 2 sets of credentials:

* OpenStack credential (to talk to the API)
* Machine credential (to SSH into the instantiated VMs)

First let's start with the OpenStack credential. Select the _Credential Type_
of OpenStack. Enter your credential details such as username, password, host,
and project name.

Next, add a Machine type credential. We'll use this credential in our job
template to authenticate to the instantiated VMs. Setup the username you're
going to SSH as, and add a private SSH key that can authenticate against your
new VMs. You'll need to make sure the VMs you spin up use the public key that
matches this public key. Setting up the keys on your OpenStack instance is to
be done manually, and is out of scope for this playbook (currently?).

### Create a Cloud Inventory

Select the Inventory tab on the left, and create a new Inventory. I'm using the
RDO Cloud, so I've called my inventory _RDO Cloud_.

Once you've saved the Inventory, click on the Sources tab. Click on the _Add
Source_ button, and select _OpenStack_ from the _Source_ drop down. Name it
something like _RDO Cloud Source_ and select _RDO Cloud_ as your _Credential_.

### Create a Local Inventory

You'll also need a local inventory that gets consumed by the playbooks. First,
create a new inventory from the Inventory page via the _Add_ button drop down.I
called mine _Local Inventory_.

Once you _Save_ the inventory, then click the _Hosts_ tab, and click on the
_Add Host_ button. In the _Host Name_ field, enter `localhost`. You'll also
need to fill in a bunch of variables so that Ansible knows how to connect to
the OpenStack service when running the `os_server` module.

Example:

```
---
ansible_connection: local
cloud_name_prefix: AWX
cloud_name: mycloud
cloud_region_name: regionOne
cloud_availability_zone: nova
cloud_image: 42a43956-a445-47e5-89d0-593b9c7b07d0
cloud_flavor: m1.medium
cloud_key_name: rdo-awx
```

### Create a Job Template

Click on the Templates section, and create a new Job Template. Name the job
something like `openstack-provisioning-job`. Use a Local inventory that simply
connects to `localhost` (you could reuse the Demo inventory and credentials if
you wanted).

Under the Project section, select your `openstack-builder` project that you
created in the previous step. Then select the `provision.yml` playbook for this
job template.

In the Extra Variables section, define your cloud credentials and your instance
list (the servers you wish to be instantiated in the cloud).

Below is a sample set of values to instantiate a Kubernetes cluster with 3
nodes and a single master.

```
---
instance_list:
  - { name: kube-master, group: master, security_groups: "default" }
  - { name: kube-node-1, group: nodes, security_groups: "default" }
  - { name: kube-node-2, group: nodes, security_groups: "default" }
  - { name: kube-node-3, group: nodes, security_groups: "default" }
my_auth_url: https://cloud.project.tld:13000/v2.0
my_username: jimmysmith
my_password: sUpEr-SeCr3t-p@ssw0rD
my_project_name: "my_project"
```
