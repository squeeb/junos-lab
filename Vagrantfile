# -*- mode: ruby -*-
# vi: set ft=ruby :

# ge-0/0/0.0 defaults to NAT for SSH + management connectivity
# over Vagrant's forwarded ports.  This should configure ge-0/0/1.0
# through ge-0/0/7.0 on VirtualBox.

######### WARNING: testing only! #########
######### WARNING: testing only! #########
######### WARNING: testing only! #########
#
# this Vagrantfile can and will wreak havoc on your VBox setup, so please
# use the Vagrant boxes at https://atlas.hashicorp.com/juniper unless you're
# attempting to extend this plugin (and can lose your VBox network config)
# TODO: launch VMs from something other than travis to CI all features
#
# Note: VMware can't name interfaces, but also supports 10 interfaces
# (through ge-0/0/9.0), so you should adjust accordingly to test
#
# Note: interface descriptions in Junos don't work yet, but you woud set them
# here with 'description:'.
#
Vagrant.configure(2) do |config|

  boxfile = "juniper/ffp-12.1X47-D15.4-packetmode"
  box_version = ">=0"

  boxes = [
    {
      name: 'vsrx1',
      hostname: 'vsrx1',
      interfaces: [
        {
          description: "from vsrx3",
          private_network_ip: '192.168.98.2',
          segment: "3-1",
        },
        {
          description: "to vsrx2",
          private_network_ip: '192.168.96.1',
          segment: "1-2",
        },
      ],
    },
    {
      name: 'vsrx2',
      hostname: 'vsrx2',
      interfaces: [
        {
          description: "from vsrx1",
          private_network_ip: '192.168.96.2',
          segment: "1-2",
        },
        {
          description: "to vsrx3",
          private_network_ip: '192.168.97.1',
          segment: "2-3",
        },
      ],
    },
    {
      name: 'vsrx3',
      hostname: 'vsrx3',
      interfaces: [
        {
          description: "from vsrx2",
          private_network_ip: '192.168.97.2',
          segment: "2-3",
        },
        {
          description: "to vsrx1",
          private_network_ip: '192.168.98.1',
          segment: "3-1",
        },
      ],
    },
  ]

  boxes.each do |box|
    config.vm.define box.fetch(:name) do |c|
      c.vm.box = boxfile
      c.vm.box_version = box_version
      c.vm.hostname = box.fetch(:hostname)
      c.vm.provider :virtualbox do |vb|
        vb.memory = 2048
        vb.cpus = 2
        vb.gui = false
        vb.customize ["modifyvm", :id, "--uart1", "0x03f8", "4"]               # set a console output to /dev/null to prevent slow boot.
        vb.customize ["modifyvm", :id, "--uartmode1", "file", "/dev/null"]     #
      end

      c.vm.network :private_network, nic_type: 'virtio', virtualbox__intnet: 'intnet'

      intf = 2
      box.fetch(:interfaces).each do |interface|
        c.vm.network :private_network, ip: interface.fetch(:private_network_ip),
          nic_type: 'virtio',
          virtualbox__intnet: "intnet_srx_#{interface.fetch(:segment)}",
          :adapter => intf

        c.vm.provider :virtualbox do |vb|
          vb.customize ["modifyvm", :id, "--nicpromisc#{intf}", "allow-all"]
        end
        intf += 1
      end

    end
  end end
