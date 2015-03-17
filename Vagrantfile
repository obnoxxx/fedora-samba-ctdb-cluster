# -*- mode: ruby -*-
# vi: ft=ruby:et:ts=2:sts=2:sw=2

VAGRANTFILE_API_VERSION = "2"

require 'yaml'

#
# Defaults for Configuration data.
# Will be overridden from the settings file
# and (possibly later) from commandline parameters.
#

net_default = {
  :type   => 'veth',
  :flags  => 'up',
  :hwaddr => '',
  :name   => '',
  :ipv4   => '',
  :ipv6   => '',
}

network_opts = [ :type, :link, :flags, :hwaddr, :name, :ipv4, :ipv6 ]

ctdb = {
  :nodes_ips => [
    '172.16.1.201',
    '172.16.1.202',
    '172.16.1.203',
  ],
  :public_ips => [
    "10.111.222.211/24 eth2",
    "10.111.222.212/24 eth2",
    "10.111.222.213/24 eth2",
  ]
}

vms = [
  {
    :hostname => 'node1',
    :box => 'obnox/fedora21-64-lxc',
    :container_name => 'fedora-cluster-node1',
    :networks => [
      {
        :link => 'virbr1',
        :ipv4 => ctdb[:nodes_ips][0],
      },
      {
        :link => 'virbr2',
        #:ipv4 => '10.111.222.201',
      },
    ],
  },
  {
    :hostname => 'node2',
    :box => 'obnox/fedora21-64-lxc',
    :container_name => 'fedora-cluster-node2',
    :networks => [
      {
        :link => 'virbr1',
        :ipv4 => ctdb[:nodes_ips][1],
      },
      {
        :link => 'virbr2',
        #:ipv4 => '10.111.222.202',
      },
    ],
  },
  {
    :hostname => 'node3',
    :box => 'obnox/fedora21-64-lxc',
    :container_name => 'fedora-cluster-node3',
    :networks => [
      {
        :link => 'virbr1',
        :ipv4 => ctdb[:nodes_ips][2],
      },
      {
        :link => 'virbr2',
        #:ipv4 => '10.111.222.203',
      },
    ],
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

  if settings[:vms].is_a?(Array)
    vms = settings[:vms]
  end
  if settings[:ctdb].is_a?(Hash)
    ctdb = settings[:ctdb]
  end
  puts "Loaded settings from #{f}."
end

# TODO(?): ARGV-processing

settings = {
  :ctdb => ctdb,
  :vms  => vms,
}

File.open(f, 'w') do |file|
  file.write settings.to_yaml
end
puts "Wrote settings to #{f}."

# apply net defaults:

vms.each do |vm|
  vm[:networks].each do |net|
    net_default.keys.each do |key|
      if not net.has_key?(key)
        net[key] = net_default[key]
      end
    end
  end
end

#
# Provisioning scripts
#

PROVISION_SCRIPT = <<SCRIPT
set -e 

BACKUP_SUFFIX=".orig.$(date +%Y%m%d-%H%M%S)"

# install software

yum -y makecache fast

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
#{ctdb[:nodes_ips].join("\n")}
EOF

# create public_addresses file:
#
PUBLIC_ADDRESSES_FILE=/etc/ctdb/public_addresses
test -f ${PUBLIC_ADDRESSES_FILE} || touch ${PUBLIC_ADDRESSES_FILE}
mv -f ${PUBLIC_ADDRESSES_FILE} ${PUBLIC_ADDRESSES_FILE}${BACKUP_SUFFIX}
cat <<EOF > ${PUBLIC_ADDRESSES_FILE}
#{ctdb[:public_ips].join("\n")}
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

        machine[:networks].each do |net|
          network_opts.each do |key|
            if not net[key] == ''
             lxc.customize "network.#{key}", net[key]
            end
          end
        end
      end
      node.vm.synced_folder "shared/", "/shared"
    end
  end

end
