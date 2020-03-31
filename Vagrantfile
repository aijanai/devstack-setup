# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "centos/7"

  config.ssh.private_key_path = ["~/.ssh/id_rsa", "~/.vagrant.d/insecure_private_key"]
  config.ssh.insert_key = false
  config.vm.provision "file", source: "~/.ssh/id_rsa.pub", destination: "~/.ssh/authorized_keys"
  config.vm.provision "file", source: "~/.ssh/id_rsa", destination: "~/.ssh/id_rsa"

  config.vm.define :master do |server|
    server.vm.provider :virtualbox do |vb|
          vb.customize ["modifyvm", :id, "--memory", "5000"]
          vb.customize ["modifyvm", :id, "--cpus", "4"]
          #vb.gui = true
    end
    server.vm.network :private_network, ip: "192.168.42.101"
    server.vm.network :private_network, ip: "192.168.42.102"
  end

end
