# -*- mode: ruby -*-
# vi: set ft=ruby :

CONTROLLER_IP_START=10
WORKER_IP_START=50
NETWORK="192.168.56."
NUM_CONTROLLER_NODES=1
NUM_WORKER_NODES=2

Vagrant.configure("2") do |config|
  config.vm.provision "shell", inline: <<-SHELL

    for i in $(seq 1 #{NUM_CONTROLLER_NODES})
    do
      sed -i "/^.*controller-${i}$/d" /etc/hosts
      echo "#{NETWORK}$((#{CONTROLLER_IP_START}+i))  controller-${i}" >> /etc/hosts
    done

    for i in $(seq 1 #{NUM_WORKER_NODES})
    do
      sed -i "/^.*worker-${i}$/d" /etc/hosts
      echo "#{NETWORK}$((#{WORKER_IP_START}+i))  worker-${i}" >>  /etc/hosts
    done
    
  SHELL
  config.vm.box = "ubuntu/focal64"
  config.vm.box_check_update = false

  (1..NUM_CONTROLLER_NODES).each do |idx|
    config.vm.define "controller-#{idx}" do |controller|
      controller.vm.hostname = "controller-#{idx}"
      controller.vm.network "private_network", 
        ip: NETWORK + "#{CONTROLLER_IP_START + idx}"

      controller.vm.provision "shell", inline: <<-SHELL
	    echo "nameserver 8.8.8.8" >> /etc/resolv.conf
	    echo "nameserver 8.8.4.4" >> /etc/resolv.conf
        curl -LO https://go.dev/dl/go1.18.2.linux-amd64.tar.gz
        rm -rf /usr/local/go && tar -C /usr/local -xzf go1.18.2.linux-amd64.tar.gz
        echo "export PATH=\$PATH:/usr/local/go/bin" >> /etc/profile
      SHELL
      
      controller.vm.provider "virtualbox" do |vb|
        vb.gui = false
        vb.memory = "2048"
        vb.cpus = 2
      end    
    end
  end
  

  (1..NUM_WORKER_NODES).each do |i|
    config.vm.define "worker-#{i}" do |node|
      node.vm.hostname = "worker-#{i}"
      node.vm.network "private_network", ip: NETWORK + "#{WORKER_IP_START + i}"

      node.vm.provider "virtualbox" do |vb|
          vb.memory = 2048
          vb.cpus = 2
          # vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      end
      # node.vm.provision "shell", path: "scripts/common.sh"
      # node.vm.provision "shell", path: "scripts/node.sh"
    end
  end
  

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
#   config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
end
