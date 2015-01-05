# Samba/CTDB-Cluster without a clustered FS

This repository contains a Vagrantfile that describes the
setup of a Samba-CTDB-Cluster of fedora-LXC nodes without
clustered file system, using a shared host folder instead.

This is useful for testing/development of clustered Samba
and CTDB on a real cluster, but taking the complexity of
a clustered File system out of the equation.

In short:

* setup controlled by vagrant
* LXC-Containers as basis
* node OS: Fedora 21
* shared folder instead of clustered filesystem

## Prerequisites

* Host with LXC installed.

## Configuration

* Possibly prepare bridge network interfaces on the host. The example data uses libvirt's virbr interfaces.
* Edit the configuration at thet top of the Vagrantfile:
 * global network settings
 * the VMS array adapt the number of nodes and the IP configuration of each node

## Running

After adjusting the configuration, `vagrant up` will bring up the full
cluster with ctdb running on all nodes. `vagrant ssh node1` will ssh
into node1, etc.

## TODO

- configure password-less ssh between the nodes (for root)
- do provisioning with [ansible](https://github.com/ansible/ansible)
- provision ctdb public addresses
- install and initially configure samba

## Author

Michael Adam (obnox at samba dot org)
