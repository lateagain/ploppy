# -*- mode: ruby -*-
# vi: set ft=ruby :

NETWORKS = {
    "net1" => "192.168.33.",
    "net2" => "192.168.34.",
    "net3" => "192.168.35.",
}

Vagrant.configure(2) do |config|
  config.vm.box = "deb/jessie-amd64"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "256"
  end

  NETWORKS.each do |(netname, baseaddr)|
    [1, 2, 3].each do |boxnumber|
      boxname = "ploppy_#{netname}_box#{boxnumber}"
  
      config.vm.define boxname do |pb|
        config.vm.provider "virtualbox" do |vb|
          vb.name = boxname
        end
        pb.vm.network "private_network", ip: baseaddr + "#{boxnumber}",
          virtualbox__intnet: "ploppy_#{netname}"
      end
    end
  end

  config.vm.define "ploppy_net1_net2_bridge" do |pb|
    config.vm.provider "virtualbox" do |vb|
      vb.name = "ploppy_net1_net2_bridge"
    end
    pb.vm.network "private_network", ip: "192.168.33.4",
      virtualbox__intnet: "ploppy_net1"
    pb.vm.network "private_network", ip: "192.168.34.5",
      virtualbox__intnet: "ploppy_net2"
  end

  config.vm.define "ploppy_net2_net3_bridge" do |pb|
    config.vm.provider "virtualbox" do |vb|
      vb.name = "ploppy_net2_net3_bridge"
    end
    pb.vm.network "private_network", ip: "192.168.34.4",
      virtualbox__intnet: "ploppy_net2"
    pb.vm.network "private_network", ip: "192.168.35.5",
      virtualbox__intnet: "ploppy_net3"
  end
    
  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   sudo apt-get update
  #   sudo apt-get install -y apache2
  # SHELL
end
