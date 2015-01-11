# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

#
# Configuration data:
# Adjust these to your needs.
#

INTERNAL_NET_LINK = 'virbr1'
INTERNAL_NET_TYPE = 'veth'

PUBLIC_NET_TYPE = "veth"
PUBLIC_NET_LINK = "virbr2"

PUBLIC_IPS = [
  "10.111.222.211/24 eth2",
  "10.111.222.212/24 eth2",
  "10.111.222.213/24 eth2",
]

VMS = [
  {
    :hostname => 'node1',
    :container_name => 'fedora-cluster-node1',
    :internal_ip => '172.16.1.201',
    :public_ip => '10.111.222.201',
  },
  {
    :hostname => 'node2',
    :container_name => 'fedora-cluster-node2',
    :internal_ip => '172.16.1.202',
    :public_ip => '10.111.222.202',
  },
  {
    :hostname => 'node3',
    :container_name => 'fedora-cluster-node3',
    :internal_ip => '172.16.1.203',
    :public_ip => '10.111.222.203',
  },
]


#
# Provisioning scripts
#

PROVISION_SCRIPT = <<SCRIPT
set -e 

BACKUP_SUFFIX=".orig.$(date +%Y%m%d-%H%M%S)"

# install software
#
# dependencies needed due to missing Requires in ctdb:
yum -y install ethtool net-tools
yum -y install ctdb samba
# debug tools
yum -y install valgrind strace gdb

systemctl stop ctdb.service

# create nodes file:
#
NODES_FILE="/etc/ctdb/nodes"
test -f ${NODES_FILE} || touch ${NODES_FILE}
mv -f ${NODES_FILE} ${NODES_FILE}${BACKUP_SUFFIX}
cat <<EOF > ${NODES_FILE}
#{VMS.map { |vm| vm[:internal_ip] }.join("\n")}
EOF

# create public_addresses file:
#
PUBLIC_ADDRESSES_FILE=/etc/ctdb/public_addresses
test -f ${PUBLIC_ADDRESSES_FILE} || touch ${PUBLIC_ADDRESSES_FILE}
mv -f ${PUBLIC_ADDRESSES_FILE} ${PUBLIC_ADDRESSES_FILE}${BACKUP_SUFFIX}
cat <<EOF > ${PUBLIC_ADDRESSES_FILE}
#{PUBLIC_IPS.join("\n")}
EOF

# prepare ctdb config:
#
CTDB_CONFIG_FILE=/etc/sysconfig/ctdb
test -f ${CTDB_CONFIG_FILE} || touch ${CTDB_CONFIG_FILE}
mv ${CTDB_CONFIG_FILE} ${CTDB_CONFIG_FILE}${BACKUP_SUFFIX}
mkdir -p /shared/ctdb/
cat <<EOF > ${CTDB_CONFIG_FILE}
CTDB_NODES=${NODES_FILE}
CTDB_PUBLIC_ADDRESSES=${PUBLIC_ADDRESSES_FILE}
CTDB_RECOVERY_LOCK=/shared/ctdb/reclock
CTDB_MANAGES_SAMBA="yes"
#CTDB_MANAGES_WINBIND="yes"
EOF

# configure samba
#
SMB_CONF=/etc/samba/smb.conf
test -f ${SMB_CONF} || touch ${SMB_CONF}
mv ${SMB_CONF} ${SMB_CONF}${BACKUP_SUFFIX}
cat <<EOF > ${SMB_CONF}
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
        # internal network
        lxc.customize "network.type", INTERNAL_NET_TYPE
        lxc.customize "network.link", INTERNAL_NET_LINK
        lxc.customize "network.flags", "up"
        lxc.customize "network.ipv4", machine[:internal_ip]
        # public network
        lxc.customize "network.type", PUBLIC_NET_TYPE
        lxc.customize "network.link", PUBLIC_NET_LINK
        lxc.customize "network.flags", "up"
        #lxc.customize "network.ipv4", machine[:public_ip]
      end
      node.vm.synced_folder "shared/", "/shared"
    end
  end

end
