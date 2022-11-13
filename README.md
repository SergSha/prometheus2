<h3>### PROMETHEUS ###</h3>

<p>Настройка мониторинга</p>

<h4>Описание домашнего задания</h4>

<ul>Настроить дашборд с 4-мя графиками:
<li>память;</li>
<li>процессор;</li>
<li>диск;</li>
<li>сеть.</li>
</ul>

<ul>Настроить на одной из систем:
<li>zabbix (использовать screen (комплексный экран);</li>
<li>prometheus - grafana.</li>
<li>использование систем, примеры которых не рассматривались на занятии.<br />
Список возможных систем был приведен в презентации.</li>
</ul>

<p>В качестве результата прислать скриншот экрана - дашборд должен содержать в названии имя приславшего.</p>

<h4>Создание стенда "Dynamic Web"</h4>

<p>Содержимое Vagrantfile:</p>

<pre>[user@localhost dynamic_web]$ <b>vi ./Vagrantfile</b></pre>

<pre># -*- mode: ruby -*-
# vi: set ft=ruby :

MACHINES = {
  :dynweb => {
    :box_name => "centos/7",
    :vm_name => "dynweb",
    :ip => '192.168.50.10', # for ansible
    :mem => '2048',
    :cpus => '2'
  }
}
Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
      box.vm.host_name = boxname.to_s
      box.vm.network "private_network", ip: boxconfig[:ip]
      box.vm.network "forwarded_port", guest: 8081, host: 8081
      box.vm.network "forwarded_port", guest: 8082, host: 8082
      box.vm.network "forwarded_port", guest: 8083, host: 8083
      box.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", boxconfig[:mem]]
        vb.customize ["modifyvm", :id, "--cpus", boxconfig[:cpus]]
      end
#      if boxconfig[:vm_name] == "dynweb"
#        box.vm.provision "ansible" do |ansible|
#          ansible.playbook = "ansible/playbook.yml"
#          ansible.inventory_path = "ansible/hosts"
#          ansible.become = true
#          ansible.host_key_checking = "false"
#          ansible.limit = "all"
#          ansible.verbose = "vvv"
#        end
#      end
    end
  end
end</pre>

<p>Запустим виртуальную машину:</p>

<pre>[user@localhost dynamic_web]$ <b>vagrant up</b></pre>

<pre>[user@localhost dynamic_web]$ <b>vagrant status</b>
Current machine states:

dynweb                    running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
[user@localhost dynamic_web]$</pre>

<pre>[user@localhost dynamic_web]$ <b>vagrant ssh dynweb</b>
[vagrant@dynweb ~]$ <b>sudo -i</b>
[root@dynweb ~]#</pre>

<p>мы рассмотрим вариант стенда nginx + php-fpm (wordpress) + python (django) + js(node.js) с деплоем через docker-compose.</p>
