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

<pre>[root@prometheus ~]# <b>curl -LO https://github.com/prometheus/prometheus/releases/download/v2.37.2/prometheus-2.37.2.linux-amd64.tar.gz</b>
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 79.9M  100 79.9M    0     0  7943k      0  0:00:10  0:00:10 --:--:-- 9410k
[root@prometheus ~]#</pre>

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

<pre>[root@prometheus prometheus-2.37.2.linux-amd64]# <b>cp -rf console{_libraries,s} prometheus.yml /etc/prometheus</b>
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

<pre>[root@prometheus prometheus-2.37.2.linux-amd64]# <b>systemctl daemon-reload</b>
[root@prometheus prometheus-2.37.2.linux-amd64]#</pre>

<p>Запускаем prometheus сервис и включаем автозапуск:</p>

<pre>[root@prometheus prometheus-2.37.2.linux-amd64]# <b>systemctl enable prometheus --now</b>
Created symlink from /etc/systemd/system/multi-user.target.wants/prometheus.service to /etc/systemd/system/prometheus.service.
[root@prometheus prometheus-2.37.2.linux-amd64]#</pre>

<p>Состояние запущенного prometheus сервиса:</p>

<pre>[root@prometheus prometheus-2.37.2.linux-amd64]# <b>systemctl status prometheus</b>
● prometheus.service - Prometheus Monitoring
   Loaded: loaded (/etc/systemd/system/prometheus.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2022-11-14 17:26:43 UTC; 43s ago
 Main PID: 3387 (prometheus)
   CGroup: /system.slice/prometheus.service
           └─3387 /usr/local/bin/prometheus --config.file /etc/prometheus/pro...

Nov 14 17:26:43 prometheus prometheus[3387]: ts=2022-11-14T17:26:43.240Z cal..."
Nov 14 17:26:43 prometheus prometheus[3387]: ts=2022-11-14T17:26:43.241Z cal...e
Nov 14 17:26:43 prometheus prometheus[3387]: ts=2022-11-14T17:26:43.241Z cal...0
Nov 14 17:26:43 prometheus prometheus[3387]: ts=2022-11-14T17:26:43.241Z cal…7ms
Nov 14 17:26:43 prometheus prometheus[3387]: ts=2022-11-14T17:26:43.242Z cal...C
Nov 14 17:26:43 prometheus prometheus[3387]: ts=2022-11-14T17:26:43.242Z cal..."
Nov 14 17:26:43 prometheus prometheus[3387]: ts=2022-11-14T17:26:43.242Z cal...l
Nov 14 17:26:43 prometheus prometheus[3387]: ts=2022-11-14T17:26:43.244Z call…µs
Nov 14 17:26:43 prometheus prometheus[3387]: ts=2022-11-14T17:26:43.244Z cal..."
Nov 14 17:26:43 prometheus prometheus[3387]: ts=2022-11-14T17:26:43.244Z cal..."
Hint: Some lines were ellipsized, use -l to show in full.
[root@prometheus prometheus-2.37.2.linux-amd64]#</pre>

<p>Возвращаемся к домашнему директорию:</p>

<pre>[root@prometheus prometheus-2.37.2.linux-amd64]# <b>cd</b>
[root@prometheus ~]#</pre>

<p>Удаляем директорий <i>prometheus-2.37.2.linux-amd64</i>:</p>

<pre>[root@prometheus ~]# <b>rm -rf ./prometheus-2.37.2.linux-amd64</b></pre>

<h4>Установка Grafana</h4>

<p>Скачиваем дистрибутив <i>grafana</i>:</p>

<pre>[root@prometheus ~]# <b>curl -LO https://dl.grafana.com/oss/release/grafana-9.2.4-1.x86_64.rpm</b>
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 91.3M  100 91.3M    0     0  6982k      0  0:00:13  0:00:13 --:--:-- 8210k
[root@prometheus ~]#</pre>

<pre>[root@prometheus ~]# <b>ls -l ./grafana-9.2.4-1.x86_64.rpm</b>
-rw-r--r--. 1 root root 95785537 Nov 14 17:28 ./grafana-9.2.4-1.x86_64.rpm
[root@prometheus ~]#</pre>

<p>Устанавливаем <i>grafana</i>:</pre>

<pre>[root@prometheus ~]# <b>yum install ./grafana-9.2.4-1.x86_64.rpm -y</b></pre>

<p>Перечитываем конфигурацию systemd:</p>

<pre>[root@prometheus ~]# <b>systemctl daemon-reload</b>
[root@prometheus ~]#</pre>

<p>Запускаем grafana-server сервис и включаем автозапуск:</p>

<pre>[root@prometheus ~]# <b>systemctl enable grafana-server --now</b>
Created symlink from /etc/systemd/system/multi-user.target.wants/grafana-server.service to /usr/lib/systemd/system/grafana-server.service.
[root@prometheus ~]#</pre>

<p>Состояние запущенного grafana-server сервиса:</p>

<pre>[root@prometheus ~]# <b>systemctl status grafana-server</b>
● grafana-server.service - Grafana instance
   Loaded: loaded (/usr/lib/systemd/system/grafana-server.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2022-11-14 17:35:18 UTC; 43s ago
     Docs: http://docs.grafana.org
 Main PID: 22362 (grafana-server)
   CGroup: /system.slice/grafana-server.service
           └─22362 /usr/sbin/grafana-server --config=/etc/grafana/grafana.ini...

Nov 14 17:35:18 prometheus grafana-server[22362]: logger=infra.usagestats.col...
Nov 14 17:35:18 prometheus grafana-server[22362]: logger=server t=2022-11-14T...
Nov 14 17:35:18 prometheus grafana-server[22362]: logger=provisioning.alertin...
Nov 14 17:35:18 prometheus grafana-server[22362]: logger=provisioning.alertin...
Nov 14 17:35:18 prometheus systemd[1]: Started Grafana instance.
Nov 14 17:35:18 prometheus grafana-server[22362]: logger=http.server t=2022-1...
Nov 14 17:35:18 prometheus grafana-server[22362]: logger=ngalert t=2022-11-14...
Nov 14 17:35:18 prometheus grafana-server[22362]: logger=grafanaStorageLogger...
Nov 14 17:35:18 prometheus grafana-server[22362]: logger=ticker t=2022-11-14T...
Nov 14 17:35:18 prometheus grafana-server[22362]: logger=ngalert.multiorg.ale...
Hint: Some lines were ellipsized, use -l to show in full.
[root@prometheus ~]#</pre>

<p>Удаляем архив <i>grafana-9.2.4-1.x86_64.rpm</i>:</p>

<pre>[root@prometheus ~]# <b>rm -rf ./grafana-9.2.4-1.x86_64.rpm</b>
[root@prometheus ~]#</pre>

<h4>Установка node_exporter</h4>

<p>Скачиваем дистрибутив node_exporter:</p>

<pre>[root@prometheus ~]# <b>curl -LO https://github.com/prometheus/node_exporter/releases/download/v1.4.0/node_exporter-1.4.0.linux-amd64.tar.gz</b>
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 9874k  100 9874k    0     0  4460k      0  0:00:02  0:00:02 --:--:-- 8002k
[root@prometheus ~]#</pre>

<pre>[root@prometheus ~]# <b>ls -l ./node_exporter-1.4.0.linux-amd64.tar.gz</b>
-rw-r--r--. 1 root root 10111972 Nov 14 17:40 ./node_exporter-1.4.0.linux-amd64.tar.gz
[root@prometheus ~]#</pre>

<p>Распакуем скачанный архив <i>node_exporter-1.4.0.linux-amd64.tar.gz</i>:</p>

<pre>[root@prometheus ~]# <b>tar -zxf ./node_exporter-1.4.0.linux-amd64.tar.gz</b></pre>

<pre>[root@prometheus ~]# <b>ls -l</b>
total 9892
...
drwxr-xr-x. 2 3434 3434       56 Sep 26 12:39 node_exporter-1.4.0.linux-amd64
-rw-r--r--. 1 root root 10111972 Nov 14 17:40 node_exporter-1.4.0.linux-amd64.tar.gz
...
[root@prometheus ~]#</pre>

<p>За дальнейшей ненадобностью удаляем скачанный архив <i>node_exporter-1.4.0.linux-amd64.tar.gz</i>:</p>

<pre>[root@prometheus ~]# <b>rm -f ./node_exporter-1.4.0.linux-amd64.tar.gz</b>
[root@prometheus ~]#</pre>

<p>Перейдём в каталог с распакованными файлами:</p>

<pre>[root@prometheus ~]# <b>cd ./node_exporter-1.4.0.linux-amd64</b>
[root@prometheus node_exporter-1.4.0.linux-amd64]#</pre>

<pre>[root@prometheus node_exporter-1.4.0.linux-amd64]# <b>ls -l</b>
total 19200
-rw-r--r--. 1 3434 3434    11357 Sep 26 12:39 LICENSE
-rw-r--r--. 1 3434 3434      463 Sep 26 12:39 NOTICE
-rwxr-xr-x. 1 3434 3434 19640886 Sep 26 12:33 node_exporter
[root@prometheus node_exporter-1.4.0.linux-amd64]#</pre>

<p>Скопируем исполняемый файл <i>node_exporter</i> в /usr/local/bin:</p>

<pre>[root@prometheus node_exporter-1.4.0.linux-amd64]# <b>cp -f ./node_exporter /usr/local/bin/</b>
[root@prometheus node_exporter-1.4.0.linux-amd64]#</pre>

<p>Создаём системного пользователя <i>node_exporter</i>, от которого будем запускать сборщика метрик:</p>

<pre>[root@prometheus node_exporter-1.4.0.linux-amd64]# <b>useradd --no-create-home --shell /bin/false node_exporter</b>
[root@prometheus node_exporter-1.4.0.linux-amd64]#</pre>

<p>Задаем владельца для исполняемого файла:</p>

<pre>[root@prometheus node_exporter-1.4.0.linux-amd64]# <b>chown -R node_exporter: /usr/local/bin/node_exporter</b>
[root@prometheus node_exporter-1.4.0.linux-amd64]#</pre>

<p>Создаем файл node_exporter.service в systemd:</p>

<pre>[root@prometheus node_exporter-1.4.0.linux-amd64]# <b>vi /etc/systemd/system/node_exporter.service</b>
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
Created symlink from /etc/systemd/system/multi-user.target.wants/node_exporter.service to /etc/systemd/system/node_exporter.service.
[root@prometheus node_exporter-1.4.0.linux-amd64]#</pre>

<p>Состояние запущенного node_exporter сервиса:</p>

<pre>[root@prometheus node_exporter-1.4.0.linux-amd64]# <b>systemctl status node_exporter</b>
● node_exporter.service - Node Exporter Service
   Loaded: loaded (/etc/systemd/system/node_exporter.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2022-11-14 17:56:15 UTC; 30s ago
 Main PID: 22440 (node_exporter)
   CGroup: /system.slice/node_exporter.service
           └─22440 /usr/local/bin/node_exporter

Nov 14 17:56:15 prometheus node_exporter[22440]: ts=2022-11-14T17:56:15.474Z...e
Nov 14 17:56:15 prometheus node_exporter[22440]: ts=2022-11-14T17:56:15.474Z...e
Nov 14 17:56:15 prometheus node_exporter[22440]: ts=2022-11-14T17:56:15.474Z...x
Nov 14 17:56:15 prometheus node_exporter[22440]: ts=2022-11-14T17:56:15.474Z...s
Nov 14 17:56:15 prometheus node_exporter[22440]: ts=2022-11-14T17:56:15.474Z...e
Nov 14 17:56:15 prometheus node_exporter[22440]: ts=2022-11-14T17:56:15.474Z...t
Nov 14 17:56:15 prometheus node_exporter[22440]: ts=2022-11-14T17:56:15.474Z...s
Nov 14 17:56:15 prometheus node_exporter[22440]: ts=2022-11-14T17:56:15.474Z...s
Nov 14 17:56:15 prometheus node_exporter[22440]: ts=2022-11-14T17:56:15.474Z...0
Nov 14 17:56:15 prometheus node_exporter[22440]: ts=2022-11-14T17:56:15.474Z...e
Hint: Some lines were ellipsized, use -l to show in full.
[root@prometheus node_exporter-1.4.0.linux-amd64]#</pre>

<p>Возвращаемся к домашнему директорию:</p>

<pre>[root@prometheus node_exporter-1.4.0.linux-amd64]# <b>cd</b>
[root@prometheus ~]#</pre>

<p>Удаляем директорий <i>node_exporter-1.4.0.linux-amd64</i>:</p>

<pre>[root@prometheus ~]# <b>rm -rf ./node_exporter-1.4.0.linux-amd64</b>
[root@prometheus ~]#</pre>

<p>Открываем веб-браузер и переходим по адресу http://192.168.50.10:9100/metrics — мы увидим метрики, собранные node_exporter:</p>

<img src="./screens/Screenshot from 2022-11-14 20-59-21.png" alt="node_expoter_metrics" border="1" />

<h4>Отображение метрик с node_exporter в консоли prometheus</h4>

<p>Открываем конфигурационный файл <i>prometheus.yml</i>:</p>

<pre>[root@prometheus ~]# <b>vi /etc/prometheus/prometheus.yml</b>
[root@prometheus ~]#</pre>

<p>В разделе scrape_configs добавим:</p>

<pre>scrape_configs:
  ...
  - job_name: 'node_exporter_clients'
    scrape_interval: 5s
    static_configs:
      - targets: ['192.168.50.10:9100']</pre>

<p>Чтобы настройка вступила в действие, перезагружаем наш prometheus сервис:</p>

<pre>[root@prometheus ~]# <b>systemctl restart prometheus</b>
[root@prometheus ~]#</pre>

<p>В веб-браузере переходим по адресу http://192.168.50.10:3000:</p>

<img src="./screens/Screenshot from 2022-11-14 21-51-18.png" alt="Grafana" border="1" />

<p>Для авторизации используем логин и пароль: admin / admin.</p>

<img src="./screens/Screenshot from 2022-11-14 21-56-41.png" alt="Grafana" border="1" />

<p>Система может потребовать задать новый пароль — вводим его дважды: Otus1234 / Otus1234.</p>

<img src="./screens/Screenshot from 2022-11-14 21-58-31.png" alt="Grafana" border="1" />

<p>[Configuration] - [Data sources]</p>

<img src="./screens/Screenshot from 2022-11-14 22-05-16.png" alt="Grafana" border="1" />

<img src="./screens/Screenshot from 2022-11-14 22-08-05.png" alt="Grafana" border="1" />

<p>[Add data source]</p>

<img src="./screens/Screenshot from 2022-11-14 22-09-12.png" alt="Grafana" border="1" />

<p>[Prometheus]</p>

<img src="./screens/Screenshot from 2022-11-14 22-10-37.png" alt="Grafana" border="1" />

<p>В поле URL вводим http://192.168.50.10:9090</p>

<img src="./screens/Screenshot from 2022-11-14 22-21-30.png" alt="Grafana" border="1" />

<p>Спускаемся вниз до конца</p>

<img src="./screens/Screenshot from 2022-11-14 22-26-46.png" alt="Grafana" border="1" />

<p>[Save & test]</p>

<img src="./screens/Screenshot from 2022-11-14 22-29-21.png" alt="Grafana" border="1" />

<p>Затем зайдём на сайт Grafana по ссылке https://grafana.com/grafana/dashboards/</p>

<img src="./screens/Screenshot from 2022-11-14 22-31-57.png" alt="Grafana" border="1" />

<p>В поле поиска вводим node exporter</p>

<img src="./screens/Screenshot from 2022-11-14 22-34-23.png" alt="Grafana" border="1" />

<p>Из результатов поиска выберем, например, Node Exporter Full</p>

<img src="./screens/Screenshot from 2022-11-14 22-39-17.png" alt="Grafana" border="1" />

<p>Видим, что код 1860. Скопируем это значение или кликаем по Copy ID to Clipboard</p>

<img src="./screens/Screenshot from 2022-11-14 22-40-49.png" alt="Grafana" border="1" />

<p>Возвращаемся к своей вкладке и в меню [Dashboards] кликаем [+ Import]</p>

<img src="./screens/Screenshot from 2022-11-14 22-43-01.png" alt="Grafana" border="1" />

<p>В поле "Import via grafana.com" вводим "1860" и нажимаем [Load]</p>

<img src="./screens/Screenshot from 2022-11-14 22-47-59.png" alt="Grafana" border="1" />

<img src="./screens/Screenshot from 2022-11-14 22-49-25.png" alt="Grafana" border="1" />

<p>Внизу в поле выбираем Prometheus</p>

<img src="./screens/Screenshot from 2022-11-14 22-51-10.png" alt="Grafana" border="1" />

<p>[Import]</p>

<img src="./screens/Screenshot from 2022-11-14 22-53-57.png" alt="Grafana" border="1" />

<p>Спустя некоторое время видим изменения.</p>

<img src="./screens/Screenshot from 2022-11-14 23-08-54.png" alt="Grafana" border="1" />

<h4>Запуск стенда "Prometheus"</h4>

<p>Запустить стенд с помощью следующей команды:</p>

<pre>$ git clone https://github.com/SergSha/prometheus2.git && cd ./prometheus2/ && vagrant up</pre>

<p>После завершения открываем веб-браузер и в адресной строке вводим:<br />

<pre>localhost:3000</pre>

<p>Откроется стартовая страница Grafana. В случае неудачи открываем по <a href="http://192.168.50.10:3000">ссылке</a>.</p>

<p>Для авторизации используем логин и пароль: admin / admin.</p>

<p>Система может потребовать задать новый пароль — вводим его дважды.</p>

<p>В меню [Configuration] кликаем [Data sources].</p>

<p>Нажимаем [Add data source]</p>

<p>Выбираем "Prometheus"</p>

<p>В поле URL вводим http://192.168.50.10:9090</p>

<p>Спускаемся вниз до конца и нажимаем [Save & test]</p>

<p>В меню [Dashboards] кликаем [+ Import]</p>

<p>В поле "Import via grafana.com" вводим "1860" и нажимаем [Load]</p>

<p>Внизу в поле выбираем "Prometheus" и нажимаем [Import].</p>

<p>Должен получиться дашборд с необходимыми графиками.</p>


