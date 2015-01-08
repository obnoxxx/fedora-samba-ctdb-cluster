# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

#
# Configuration data:
# Adjust these to your needs.
#

INTERNAL_NET_LINK = 'virbr1'
INTERNAL_NET_TYPE = 'veth'

VMS = [
  {
    :hostname => 'node1',
    :container_name => 'fedora-cluster-node1',
    :internal_ip => '172.16.1.201',
  },
  {
    :hostname => 'node2',
    :container_name => 'fedora-cluster-node2',
    :internal_ip => '172.16.1.202',
  },
  {
    :hostname => 'node3',
    :container_name => 'fedora-cluster-node3',
    :internal_ip => '172.16.1.203',
  },
]


#
# Provisioning scripts
#

PROVISION_SCRIPT = <<SCRIPT
set -e 

yum -y install ctdb samba

# create nodes file:
echo #{VMS[0][:internal_ip]} >  /etc/ctdb/nodes
echo #{VMS[1][:internal_ip]} >> /etc/ctdb/nodes
echo #{VMS[2][:internal_ip]} >> /etc/ctdb/nodes

# prepare ctdb config:
mv /etc/sysconfig/ctdb /etc/sysconfig/ctdb.orig.`date +%Y%m%d-%H%M%S`
mkdir -p /shared/ctdb/
cat <<EOF > /etc/sysconfig/ctdb
#CTDB_NODES=/etc/ctdb/nodes
CTDB_RECOVERY_LOCK=/shared/ctdb/reclock
CTDB_MANAGES_SAMBA="yes"
#CTDB_MANAGES_WINBIND="yes"
EOF

mv /etc/samba/smb.conf /etc/samba/smb.conf.orig.`date +%Y%m%d-%H%M%S`
cat <<EOF > /etc/samba/smb.conf
[global]
	netbios name = sambacluster
	workgroup = vagrant
	security = user

	clustering = yes
	#include = registry

[share1]
	path = /shared/share1
	read only = no
EOF

mkdir -p /shared/share1
chmod a+rwx /shared/share1

# prepare password-less ssh for root between nodes (for onnode):
#(TODO)

systemctl start ctdb.service
SCRIPT



#
# The vagrant machine definitions
#

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
  end

  #
  # Provisioning common to all machines:
  #
  config.vm.provision :shell, inline: PROVISION_SCRIPT

  VMS.each do |machine|
    config.vm.define machine[:hostname] do |node|
      node.vm.box = "obnox/fedora21-64-lxc"
      node.vm.hostname = machine[:hostname]
      node.vm.provider :lxc do |lxc|
        lxc.container_name = machine[:container_name]
        lxc.customize "network.type", INTERNAL_NET_TYPE
        lxc.customize "network.link", INTERNAL_NET_LINK
        lxc.customize "network.flags", "up"
        lxc.customize "network.ipv4", machine[:internal_ip]
      end
      node.vm.synced_folder "shared/", "/shared"
    end
  end

end
