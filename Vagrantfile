# -*- mode: ruby -*-
# vi: set ft=ruby :

NETWORKS = {
    "net1" => "192.168.33.",
    "net2" => "192.168.34.",
    "net3" => "192.168.33.",
}

Vagrant.configure(2) do |config|
  config.vm.box = "deb/jessie-amd64"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "512"
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
    pb.vm.network "private_network", ip: "192.168.33.5",
      virtualbox__intnet: "ploppy_net3"
  end
    
  config.vm.provision "shell", privileged: false, inline: <<-SHELL
    sudo dd if=/dev/zero of=/swapfile bs=1M count=1024
    sudo chmod 0600 /swapfile
    sudo mkswap /swapfile
    sudo bash -c 'echo /swapfile none swap defaults 0 0 >>/etc/fstab'
    sudo swapon /swapfile
    curl -L http://install.perlbrew.pl|bash
    source ~/perl5/perlbrew/etc/bashrc
    echo source ~/perl5/perlbrew/etc/bashrc >> .profile
    perlbrew init
    perlbrew install perl-5.20.2
    perlbrew switch perl-5.20.2
    cpan Alien::ZMQ
  SHELL
end
