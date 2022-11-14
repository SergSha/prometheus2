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

<pre>[root@prometheus ~]# <b>setenforce 0</b>
[root@prometheus ~]#</pre>

<pre>[root@prometheus ~]# <b>sed -i 's/^SELINUX=.*/SELINUX=permissive/g' /etc/selinux/config</b>
[root@prometheus ~]#</pre>

<h4>Установка Prometheus</h4>

<p>Скачиваем дистрибутив prometheus:</p>

<pre[root@prometheus ~]# <b>curl -LO https://github.com/prometheus/prometheus/releases/download/v2.37.2/prometheus-2.37.2.linux-amd64.tar.gz</b></pre>

<pre>[root@prometheus ~]# <b>ls -l ./prometheus-2.37.2.linux-amd64.tar.gz</b>
-rw-r--r--. 1 root root 83879445 Nov  4 11:32 ./prometheus-2.37.2.linux-amd64.tar.gz
[root@prometheus ~]#</pre>

<p>Распакуем скачанный архив <i>prometheus-2.37.2.linux-amd64.tar.gz</i>:</p>

<pre>[root@prometheus ~]# <b>tar zxf ./prometheus-2.37.2.linux-amd64.tar.gz</b>
[root@prometheus ~]#</pre>

<pre>[root@prometheus ~]# <b>ls -l</b>
total 81932
...
drwxr-xr-x. 4 3434 3434      132 Nov  4 11:28 prometheus-2.37.2.linux-amd64
-rw-r--r--. 1 root root 83879445 Nov  4 11:32 prometheus-2.37.2.linux-amd64.tar.gz
[root@prometheus ~]#</pre>

<p>За дальнейшей ненадобностью удаляем скачанный архив prometheus:</p>

<pre>[root@prometheus ~]# <b>rm -f ./prometheus-2.37.2.linux-amd64.tar.gz</b>
[root@prometheus ~]#</pre>

<p>Создаём каталоги, в которые скопируем файлы для prometheus:</p>

<pre>[root@prometheus ~]# <b>mkdir /etc/prometheus</b>
[root@prometheus ~]#</pre>

<pre>[root@prometheus ~]# <b>mkdir /var/lib/prometheus</b>
[root@prometheus ~]#</pre>

<p>Перейдём в каталог с распакованными файлами:</p>

<pre>[root@prometheus ~]# <b>cd ./prometheus-2.37.2.linux-amd64/</b>
[root@prometheus prometheus-2.37.2.linux-amd64]#</pre>

<p>Список файлов и каталогов в этом директории:</p>

<pre>[root@prometheus prometheus-2.37.2.linux-amd64]# <b>ls -l</b>
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

<pre>[root@prometheus prometheus-2.37.2.linux-amd64]# <b>cp -f prom{etheus,tool} /usr/local/bin/</b>
[root@prometheus prometheus-2.37.2.linux-amd64]#</pre>

<pre>[root@prometheus prometheus-2.37.2.linux-amd64]# <b>cp -r console{_libraries,s} prometheus.yml /etc/prometheus</b>
[root@prometheus prometheus-2.37.2.linux-amd64]#</pre>

<p>Создаём пользователя <i>prometheus</i>, от которого будем запускать систему мониторинга:</p>

<pre>[root@prometheus prometheus-2.37.2.linux-amd64]# <b>useradd --no-create-home --shell /bin/false prometheus</b>
[root@prometheus prometheus-2.37.2.linux-amd64]#</pre>

<p>Задаём владельца для каталогов, которые мы создали на предыдущем шаге:</p>

<pre>[root@prometheus prometheus-2.37.2.linux-amd64]# <b>chown -R prometheus: /etc/prometheus /var/lib/prometheus</b>
[root@prometheus prometheus-2.37.2.linux-amd64]#</pre>

<p>Задаём владельца для скопированных файлов:</p>

<pre>[root@prometheus prometheus-2.37.2.linux-amd64]# <b>chown prometheus: /usr/local/bin/prom{etheus,tool}</b>
[root@prometheus prometheus-2.37.2.linux-amd64]#</pre>

<p>Создадим prometheus systemd сервис:</p>

<pre>[root@prometheus prometheus-2.37.2.linux-amd64]# <b>vi /etc/systemd/system/prometheus.service</b>
[Unit]
Description=Prometheus Monitoring
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
--config.file /etc/prometheus/prometheus.yml \
--storage.tsdb.path /var/lib/prometheus/ \
--web.console.templates=/etc/prometheus/consoles \
--web.console.libraries=/etc/prometheus/console_libraries
ExecReload=/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target</pre>

<p>Перечитываем конфигурацию systemd:</p>

<pre>[root@prometheus prometheus-2.37.2.linux-amd64]# <b>systemctl daemon-reload</b></pre>

<p>Запускаем prometheus сервис и включаем автозапуск:</p>

<pre>[root@prometheus prometheus-2.37.2.linux-amd64]# <b>systemctl enable prometheus --now</b></pre>

<p>Состояние запущенного prometheus сервиса:</p>

<pre>[root@prometheus prometheus-2.37.2.linux-amd64]# <b>systemctl status prometheus</b></pre>

<p>Возвращаемся к домашнему директорию:</p>

<pre>[root@prometheus prometheus-2.37.2.linux-amd64]# <b>cd</b>
[root@prometheus ~]#</pre>

<p>Удаляем директорий <i>prometheus-2.37.2.linux-amd64</i>:</p>

<pre>[root@prometheus ~]# <b>rm -rf ./prometheus-2.37.2.linux-amd64</b></pre>

<h4>Установка Grafana</h4>

<p>Скачиваем дистрибутив <i>grafana</i>:</p>

<pre[root@prometheus ~]# <b>curl -LO https://dl.grafana.com/oss/release/grafana-9.2.4-1.x86_64.rpm</b></pre>

<pre>[root@prometheus ~]# <b>ls -l ./grafana-9.2.4-1.x86_64.rpm</b>
-rw-r--r--. 1 root root 83879445 Nov  4 11:32 ./grafana-9.2.4-1.x86_64.rpm
[root@prometheus ~]#</pre>

<p>Устанавливаем <i>grafana</i>:</pre>

<pre>[root@prometheus ~]# <b>yum install ./grafana-8.3.4-1.x86_64.rpm -y</b></pre>

<p>Перечитываем конфигурацию systemd:</p>

<pre>[root@prometheus ~]# <b>systemctl daemon-reload</b></pre>

<p>Запускаем grafana-server сервис и включаем автозапуск:</p>

<pre>[root@prometheus ~]# <b>systemctl enable grafana-server --now</b></pre>

<p>Состояние запущенного grafana-server сервиса:</p>

<pre>[root@prometheus ~]# <b>systemctl status grafana-server</b></pre>

<p>Удаляем архив <i>grafana-9.2.4-1.x86_64.rpm</i>:</p>

<pre>[root@prometheus ~]# <b>rm -rf ./grafana-9.2.4-1.x86_64.rpm</b></pre>

<h4>Установка node_exporter</h4>

<p>Скачиваем дистрибутив node_exporter:</p>

<pre>[root@prometheus ~]# <b>curl -LO https://github.com/prometheus/node_exporter/releases/download/v1.4.0/node_exporter-1.4.0.linux-amd64.tar.gz</b></pre>

<p>Распакуем скачанный архив <i>node_exporter-1.4.0.linux-amd64.tar.gz</i>:</p>

<pre>[root@prometheus ~]# <b>tar -xvf ./node_exporter-1.4.0.linux-amd64.tar.gz</b></pre>

<pre>[root@prometheus ~]# <b>ls -l</b>
total 81932
...
drwxr-xr-x. 4 3434 3434      132 Nov  4 11:28 node_exporter-1.4.0.linux-amd64
-rw-r--r--. 1 root root 83879445 Nov  4 11:32 node_exporter-1.4.0.linux-amd64.tar.gz
[root@prometheus ~]#</pre>

<p>За дальнейшей ненадобностью удаляем скачанный архив <i>node_exporter-1.4.0.linux-amd64.tar.gz</i>:</p>

<pre>[root@prometheus ~]# <b>rm -f ./node_exporter-1.4.0.linux-amd64.tar.gz</b>
[root@prometheus ~]#</pre>

<p>Перейдём в каталог с распакованными файлами:</p>

<pre>[root@prometheus ~]# <b>cd ./node_exporter-1.4.0.linux-amd64</b>
[root@prometheus node_exporter-1.4.0.linux-amd64]#</pre>

<pre>[root@prometheus node_exporter-1.4.0.linux-amd64]# <b>ls -l</b>
[root@prometheus node_exporter-1.4.0.linux-amd64]#</pre>

<p>Скопируем исполняемый файл <i>node_exporter</i> в /usr/local/bin:</p>

<pre>[root@prometheus node_exporter-1.4.0.linux-amd64]# <b>cp -f ./node_exporter /usr/local/bin/</b>
[root@prometheus node_exporter-1.4.0.linux-amd64]#</pre>

<p>Создаём системного пользователя <i>node_exporter</i>, от которого будем запускать сборщика метрик:</p>

<pre>[root@prometheus node_exporter-1.4.0.linux-amd64]# <b>useradd --no-create-home --shell /bin/false node_exporter</b></pre>

<p>Задаем владельца для исполняемого файла:</p>

<pre>[root@prometheus node_exporter-1.4.0.linux-amd64]# <b>chown -R node_exporter: /usr/local/bin/node_exporter</b>
[root@prometheus node_exporter-1.4.0.linux-amd64]#</pre>

<p>Создаем файл node_exporter.service в systemd:</p>

<pre>[root@prometheus node_exporter-1.4.0.linux-amd64]# <b>chown -R node_exporter.service</b>
[Unit]
Description=Node Exporter Service
Wants=network-online.target
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure

[Install]
WantedBy=multi-user.target</pre>

<p>Перечитываем конфигурацию systemd:</p>

<pre>[root@prometheus node_exporter-1.4.0.linux-amd64]# <b>systemctl daemon-reload</b>
[root@prometheus node_exporter-1.4.0.linux-amd64]#</pre>

<p>Запускаем node_exporter сервис и включаем автозапуск:</p>

<pre>[root@prometheus node_exporter-1.4.0.linux-amd64]# <b>systemctl enable node_exporter --now</b>
[root@prometheus node_exporter-1.4.0.linux-amd64]#</pre>

<p>Состояние запущенного node_exporter сервиса:</p>

<pre>[root@prometheus node_exporter-1.4.0.linux-amd64]# <b>systemctl status node_exporter</b>
[root@prometheus node_exporter-1.4.0.linux-amd64]#</pre>

<p>Возвращаемся к домашнему директорию:</p>

<pre>[root@prometheus node_exporter-1.4.0.linux-amd64]# <b>cd</b>
[root@prometheus ~]#</pre>

<p>Удаляем директорий <i>node_exporter-1.4.0.linux-amd64</i>:</p>

<pre>[root@prometheus ~]# <b>rm -rf ./node_exporter-1.4.0.linux-amd64</b>
[root@prometheus ~]#</pre>

<p>Открываем веб-браузер и переходим по адресу http://<IP-адрес сервера или клиента>:9100/metrics — мы увидим метрики, собранные node_exporter:</p>

<h4>Отображение метрик с node_exporter в консоли prometheus</h4>

<p>Открываем конфигурационный файл <i>prometheus.yml</i>:</p>

<pre>[root@prometheus ~]# <b>vi /etc/prometheus/prometheus.yml</b>
[root@prometheus ~]#</pre>

<pre>scrape_configs:
  ...
  - job_name: 'node_exporter_clients'
    scrape_interval: 5s
    static_configs:
      - targets: ['192.168.50.10:9100']</pre>

<p>Перезагружаем prometheus сервис:</p>

<pre>[root@prometheus ~]# <b>systemctl restart prometheus</b>
[root@prometheus ~]#</pre>



