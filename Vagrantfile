# -*- mode: ruby -*-
# vi: ft=ruby:et:ts=2:sts=2

VAGRANTFILE_API_VERSION = "2"

require 'yaml'

#
# Defaults for Configuration data.
# Will be overridden from the settings file
# and (possibly later) from commandline parameters.
#

internal_net_type = 'veth'
internal_net_link = 'virbr1'

public_net_type = "veth"
public_net_link = "virbr2"

public_ips = [
  "10.111.222.211/24 eth2",
  "10.111.222.212/24 eth2",
  "10.111.222.213/24 eth2",
]

vms = [
  {
    :hostname => 'node1',
    :container_name => 'fedora-cluster-node1',
    :internal_ip => '172.16.1.201',
    :public_ip => '10.111.222.201',
    :box => 'obnox/fedora21-64-lxc',

  },
  {
    :hostname => 'node2',
    :container_name => 'fedora-cluster-node2',
    :internal_ip => '172.16.1.202',
    :public_ip => '10.111.222.202',
    :box => 'obnox/fedora21-64-lxc',
  },
  {
    :hostname => 'node3',
    :container_name => 'fedora-cluster-node3',
    :internal_ip => '172.16.1.203',
    :public_ip => '10.111.222.203',
    :box => 'obnox/fedora21-64-lxc',
  },
]

#
# Load the config, if it exists,
# possibly override with commandline args,
# (currently none supported yet)
# and then store the config.
#

projectdir = File.expand_path File.dirname(__FILE__)
f = File.join(projectdir, 'vagrant.yaml')
if File.exists?(f)
  settings = YAML::load_file f
  puts "Loaded settings from #{f}."

  if settings[:internal_net_type].is_a?(String)
    internal_net_type = settings[:internal_net_type]
  end
  if settings[:internal_net_link].is_a?(String)
    internal_net_link = settings[:internal_net_link]
  end
  if settings[:public_net_type].is_a?(String)
    public_net_type = settings[:public_net_net_type]
  end
  if settings[:public_net_link].is_a?(String)
    public_net_link = settings[:public_net_net_link]
  end
  if settings[:vms].is_a?(Array)
    vms = settings[:vms]
  end
  if settings[:public_ips].is_a?(Array)
    public_ips = settings[:public_ips]
  end
end

# TODO(?): ARGV-processing

settings = {
  :internal_net_type => internal_net_type,
  :internal_net_link => internal_net_link,
  :public_net_type   => public_net_type,
  :public_net_link   => public_net_link,
  :public_ips        => public_ips,
  :vms               => vms,
}

File.open(f, 'w') do |file|
	file.write settings.to_yaml
	puts "Wrote settings to #{f}."
end

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
#{vms.map { |vm| vm[:internal_ip] }.join("\n")}
EOF

# create public_addresses file:
#
PUBLIC_ADDRESSES_FILE=/etc/ctdb/public_addresses
test -f ${PUBLIC_ADDRESSES_FILE} || touch ${PUBLIC_ADDRESSES_FILE}
mv -f ${PUBLIC_ADDRESSES_FILE} ${PUBLIC_ADDRESSES_FILE}${BACKUP_SUFFIX}
cat <<EOF > ${PUBLIC_ADDRESSES_FILE}
#{public_ips.join("\n")}
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

  vms.each do |machine|
    config.vm.define machine[:hostname] do |node|
      node.vm.box = machine[:box]
      node.vm.hostname = machine[:hostname]
      node.vm.provider :lxc do |lxc|
        lxc.container_name = machine[:container_name]
        # internal network
        lxc.customize "network.type", internal_net_type
        lxc.customize "network.link", internal_net_link
        lxc.customize "network.flags", "up"
        lxc.customize "network.ipv4", machine[:internal_ip]
        # public network
        lxc.customize "network.type", public_net_type
        lxc.customize "network.link", public_net_link
        lxc.customize "network.flags", "up"
        #lxc.customize "network.ipv4", machine[:public_ip]
      end
      node.vm.synced_folder "shared/", "/shared"
    end
  end

end
