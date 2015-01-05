# Samba/CTDB-Cluster without a clustered FS

This repository contains a Vagrantfile that describes the
setup of a Samba-CTDB-Cluster of fedora-LXC nodes without
clustered file system, using a shared host folder instead.

This is useful for testing/development of clustered Samba
and CTDB on a real cluster, but taking the complexity of
a clustered file system out of the equation.

In short:

* setup controlled by vagrant
* LXC-Containers as basis
* node OS: Fedora 21
* shared folder instead of clustered filesystem

## Prerequisites

* Linux host, attached to the network
* lxc installed
* vagrant installed
* vagrant-lxc plugin installed
* possibly preparation of bridge interfaces on the host (the example uses libvirt's `virbr` interfaces)

## Configuration

* Edit the configuration at the top of the Vagrantfile:
 * global network settings (example uses virbr interfaces)
 * the VMS array to adjust the number of nodes and the IP configuration of each node

## Running

After adjusting the configuration, `vagrant up` will bring up the full
cluster with ctdb running on all nodes. `vagrant ssh node1` will ssh
into node1, etc.

## TODO

- configure password-less ssh between the nodes (for root)
- provision ctdb public addresses
- install and initially configure samba
- do provisioning with [ansible](https://github.com/ansible/ansible)
- check whether one can spare dhcp on the default interface

## Author

Michael Adam (obnox at samba dot org)
