# -*- mode: ruby -*-
# vi: set ft=ruby :

MACHINES = {
  :prometheus => {
    :box_name => "centos/7",
    :vm_name => "prometheus",
    :ip => '192.168.50.10',
    :mem => '2048',
    :cpus => '1'
  }
}
Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
      box.vm.host_name = boxname.to_s
      box.vm.network "private_network", ip: boxconfig[:ip]
      box.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", boxconfig[:mem]]
        vb.customize ["modifyvm", :id, "--cpus", boxconfig[:cpus]]
      end
      box.vm.provision "shell", inline: <<-SHELL
        mkdir -p ~root/.ssh
        cp ~vagrant/.ssh/auth* ~root/.ssh
      SHELL
      if boxconfig[:vm_name] == "prometheus"
        box.vm.provision "ansible" do |ansible|
          ansible.playbook = "ansible/playbook.yml"
          ansible.inventory_path = "ansible/hosts"
          ansible.become = true
          ansible.host_key_checking = "false"
          ansible.limit = "all"
#          ansible.verbose = "vvv"
        end
      end
    end
  end
end
