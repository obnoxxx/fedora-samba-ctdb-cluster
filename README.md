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

## Configuration

This Vagrantfile is parametrized: The options for configuring
the number of nodes and the nodes' network interfaces and addresses
are stored in the config file 'vagrant.yml' when running any
vagrant command (except for vagrant help). The whole setup can
then conveniently be reconfigured modifying that file.

## Prerequisites

* Linux host, attached to the network (tested on Fedora 21, should work on other hosts as well)
* lxc installed
* vagrant installed
* vagrant-lxc plugin installed
* possibly preparation of bridge interfaces on the host (the example uses libvirt's `virbr` interfaces)

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
- Add mechanism to make the entier environment easily configurable.
  - E.g something like in oh-my-vagrant which uses a yaml setting file.

## Author

Michael Adam (obnox at samba dot org)
