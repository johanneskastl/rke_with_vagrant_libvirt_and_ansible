Vagrant.configure("2") do |config|

  ###################################################################################
  # define number of workers
  W = 1

  # provision W VMs as workers
  (1..W).each do |i|

    # name the VMs
    config.vm.define "rkeworker#{i}" do |node|

      # which image to use
      node.vm.box = "opensuse/Leap-15.2.x86_64"

      # sizing of the VMs
      node.vm.provider "libvirt" do |lv|
        lv.random_hostname = true
        lv.memory = 1024
        lv.cpus = 2
      end

      # set the hostname
      node.vm.hostname = "rkeworker#{i}"

      # set the IP address
      node.vm.network "private_network", ip: "192.168.123.2#{i}"

    end # config.vm.define workers

  end # each-loop workers

  ###############################################

  # define number of controlplane nodes
  M = 3

  # provision N VMs as controlplane nodes
  (1..M).each do |i|

    # name the VMs
    config.vm.define "rkeserver#{i}" do |node|

      # which image to use
      node.vm.box = "opensuse/Leap-15.2.x86_64"

      # sizing of the VMs
      node.vm.provider "libvirt" do |lv|
        lv.random_hostname = true
        lv.memory = 2048
        lv.cpus = 2
      end

      # set the hostname
      node.vm.hostname = "rkeserver#{i}"

      # set the IP address
      node.vm.network "private_network", ip: "192.168.123.1#{i}"

      # if this is the last machine
      if i == M

          #################################################
          # only target one node to not run ansible twice
          # but do not limit this to this node (set ansible.limit to all)
          node.vm.provision "ansible" do |ansible|
            ansible.compatibility_mode = "2.0"
            ansible.limit = "all"
            ansible.groups = {
              "rkeservers"  => [ "rkeserver1", "rkeserver2", "rkeserver3" ],
              "rkeworkers"  => [ "rkeworker1" ]
            }
            ansible.playbook = "ansible/playbook-vagrant.yml"
          end # node.vm.provision

      end # if-condition last machine

    end # config.vm.define controlplane nodes

  end # each-loop controlplane nodes

end # Vagrant.configure
