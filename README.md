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

<h4>Создание стенда "Prometheus"</h4>

<p>Содержимое Vagrantfile:</p>

<pre>[user@localhost prometheus2]$ <b>vi ./Vagrantfile</b></pre>

<pre># -*- mode: ruby -*-
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
#      if boxconfig[:vm_name] == "prometheus"
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

<pre>[user@localhost prometheus2]$ <b>vagrant up</b></pre>

<pre>[user@localhost prometheus2]$ vagrant status
Current machine states:

prometheus                running (virtualbox)

The VM is running. To stop this VM, you can run `vagrant halt` to
shut it down forcefully, or you can run `vagrant suspend` to simply
suspend the virtual machine. In either case, to restart it again,
simply run `vagrant up`.
[user@localhost prometheus2]$</pre>

<pre>[user@localhost prometheus2]$ <b>vagrant ssh prometheus</b>
[vagrant@prometheus ~]$ <b>sudo -i</b>
[root@prometheus ~]#</pre>

<p>Отключим selinux:</p>

<pre>[root@prometheus ~]# setenforce 0
[root@prometheus ~]#</pre>

<pre>[root@prometheus ~]# sed -i 's/^SELINUX=.*/SELINUX=permissive/g' /etc/selinux/config
[root@prometheus ~]#</pre>

<p>Установим wget:</p>

<pre>[root@prometheus ~]# yum install wget -y</pre>

<p>Скачиваем дистрибутив prometheus:</p>

<pre[root@prometheus ~]# wget https://github.com/prometheus/prometheus/releases/download/v2.37.2/prometheus-2.37.2.linux-amd64.tar.gz</pre>

<pre>[root@prometheus ~]# ls -l ./prometheus-2.37.2.linux-amd64.tar.gz 
-rw-r--r--. 1 root root 83879445 Nov  4 11:32 ./prometheus-2.37.2.linux-amd64.tar.gz
[root@prometheus ~]#</pre>

<p>Распакуем скачанный архив prometheus:</p>

<pre>[root@prometheus ~]# tar zxf ./prometheus-2.37.2.linux-amd64.tar.gz 
[root@prometheus ~]#</pre>

<pre>[root@prometheus ~]# ls -l
total 81932
...
drwxr-xr-x. 4 3434 3434      132 Nov  4 11:28 prometheus-2.37.2.linux-amd64
-rw-r--r--. 1 root root 83879445 Nov  4 11:32 prometheus-2.37.2.linux-amd64.tar.gz
[root@prometheus ~]#</pre>

<p>За дальнейшей ненадобностью удаляем скачанный архив prometheus:</p>

<pre>[root@prometheus ~]# rm -f ./prometheus-2.37.2.linux-amd64.tar.gz 
[root@prometheus ~]#</pre>

<p>Создаём каталоги, в которые скопируем файлы для prometheus:</p>

<pre>[root@prometheus ~]# mkdir /etc/prometheus
[root@prometheus ~]#</pre>

<pre>[root@prometheus ~]# mkdir /var/lib/prometheus
[root@prometheus ~]#</pre>

<p>Перейдём в каталог с распакованными файлами:</p>

<pre>[root@prometheus ~]# cd ./prometheus-2.37.2.linux-amd64/
[root@prometheus prometheus-2.37.2.linux-amd64]#</pre>

<p>Список файлов и каталогов в этом директории:</p>

<pre>[root@prometheus prometheus-2.37.2.linux-amd64]# ls -l
total 206276
-rw-r--r--. 1 3434 3434     11357 Nov  4 11:24 LICENSE
-rw-r--r--. 1 3434 3434      3773 Nov  4 11:24 NOTICE
drwxr-xr-x. 2 3434 3434        38 Nov  4 11:24 console_libraries
drwxr-xr-x. 2 3434 3434       173 Nov  4 11:24 consoles
-rwxr-xr-x. 1 3434 3434 109691493 Nov  4 11:09 prometheus
-rw-r--r--. 1 3434 3434       934 Nov  4 11:24 prometheus.yml
-rwxr-xr-x. 1 3434 3434 101509420 Nov  4 11:11 promtool
[root@prometheus prometheus-2.37.2.linux-amd64]#</pre>

<p>Распределяем файлы по каталогам:</p>

<pre>[root@prometheus prometheus-2.37.2.linux-amd64]# cp prom{etheus,tool} /usr/local/bin/
[root@prometheus prometheus-2.37.2.linux-amd64]#</pre>

<pre>[root@prometheus prometheus-2.37.2.linux-amd64]# cp -r console{_libraries,s} prometheus.yml /etc/prometheus
[root@prometheus prometheus-2.37.2.linux-amd64]#</pre>

<p>Создаём пользователя <i>prometheus</i>, от которого будем запускать систему мониторинга:</p>

<pre>[root@prometheus prometheus-2.37.2.linux-amd64]# useradd --no-create-home --shell /bin/false prometheus
[root@prometheus prometheus-2.37.2.linux-amd64]#</pre>

<p>Задаём владельца для каталогов, которые мы создали на предыдущем шаге:</p>

<pre>[root@prometheus prometheus-2.37.2.linux-amd64]# chown -R prometheus: /etc/prometheus /var/lib/prometheus
[root@prometheus prometheus-2.37.2.linux-amd64]#</pre>

<p>Задаём владельца для скопированных файлов:</p>

<pre>[root@prometheus prometheus-2.37.2.linux-amd64]# chown prometheus: /usr/local/bin/prom{etheus,tool}
[root@prometheus prometheus-2.37.2.linux-amd64]#</pre>

<p>Запускаем prometheus командой:</p>

/usr/local/bin/prometheus --config.file /etc/prometheus/prometheus.yml --storage.tsdb.path /var/lib/prometheus/ --web.console.templates=/etc/prometheus/consoles --web.console.libraries=/etc/prometheus/console_libraries







