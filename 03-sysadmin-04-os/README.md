# Домашнее задание к занятию "3.4. Операционные системы, лекция 2"

### 1. На лекции мы познакомились с [node_exporter](https://github.com/prometheus/node_exporter/releases). В демонстрации его исполняемый файл запускался в background. Этого достаточно для демо, но не для настоящей production-системы, где процессы должны находиться под внешним управлением. Используя знания из лекции по systemd, создайте самостоятельно простой [unit-файл](https://www.freedesktop.org/software/systemd/man/systemd.service.html) для node_exporter:

    * поместите его в автозагрузку,
    * предусмотрите возможность добавления опций к запускаемому процессу через внешний файл (посмотрите, например, на `systemctl cat cron`),
    * удостоверьтесь, что с помощью systemctl процесс корректно стартует, завершается, а после перезагрузки автоматически поднимается.

1. Накатил рекомендуемую [ansible роль](https://github.com/cloudalchemy/ansible-node-exporter) в виртуалке, пришлось разобраться с `vagrant provision`.
   - удалил из автозагрузки `systemctl disable node_exporter`, удостоверился что не стартует при запуске
   - добавил в автозагрузку `systemctl enable node_exporter`, удостоверился что стартует при запуске
```bash
vagrant@vagrant:~$ systemctl cat node_exporter
# /etc/systemd/system/node_exporter.service
#
# Ansible managed
#

[Unit]
Description=Prometheus Node Exporter
After=network-online.target

[Service]
Type=simple
User=node-exp
Group=node-exp
ExecStart=/usr/local/bin/node_exporter $NODE_EXPORTER_EXTRA_OPTS \
    --collector.systemd \
--collector.textfile \
    --collector.textfile.directory=/var/lib/node_exporter \
    --web.listen-address=0.0.0.0:9100 \
    --web.telemetry-path=/metrics

SyslogIdentifier=node_exporter
Restart=always
RestartSec=1
StartLimitInterval=0

ProtectHome=yes
NoNewPrivileges=yes

ProtectSystem=strict
ProtectControlGroups=true
ProtectKernelModules=true
ProtectKernelTunables=yes

[Install]
WantedBy=multi-user.target
```
2. Для дополнительных опций добавил `$NODE_EXPORTER_EXTRA_OPTS` по аналогии с юнит-файлом крона.
3. Процесс корректно завершается и стартует, в том числе после перезагрузки системы:
```bash
vagrant@vagrant:~$ systemctl start node_exporter
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===
Authentication is required to start 'node_exporter.service'.
Authenticating as: vagrant,,, (vagrant)
Password:
==== AUTHENTICATION COMPLETE ===
vagrant@vagrant:~$ systemctl status node_exporter
● node_exporter.service - Prometheus Node Exporter
     Loaded: loaded (/etc/systemd/system/node_exporter.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2021-12-06 05:08:21 UTC; 5s ago
   Main PID: 3452 (node_exporter)
      Tasks: 5 (limit: 2278)
     Memory: 3.2M
     CGroup: /system.slice/node_exporter.service
             └─3452 /usr/local/bin/node_exporter --collector.systemd --collector.textfile --collector.textfile.directory=/var/lib/node_exporter --web.listen-address=0.0.0.0:9100 --web.telemetry-path=/metri>

Dec 06 05:08:21 vagrant node_exporter[3452]: ts=2021-12-06T05:08:21.580Z caller=node_exporter.go:115 level=info collector=thermal_zone
Dec 06 05:08:21 vagrant node_exporter[3452]: ts=2021-12-06T05:08:21.580Z caller=node_exporter.go:115 level=info collector=time
Dec 06 05:08:21 vagrant node_exporter[3452]: ts=2021-12-06T05:08:21.580Z caller=node_exporter.go:115 level=info collector=timex
Dec 06 05:08:21 vagrant node_exporter[3452]: ts=2021-12-06T05:08:21.580Z caller=node_exporter.go:115 level=info collector=udp_queues
Dec 06 05:08:21 vagrant node_exporter[3452]: ts=2021-12-06T05:08:21.580Z caller=node_exporter.go:115 level=info collector=uname
Dec 06 05:08:21 vagrant node_exporter[3452]: ts=2021-12-06T05:08:21.580Z caller=node_exporter.go:115 level=info collector=vmstat
Dec 06 05:08:21 vagrant node_exporter[3452]: ts=2021-12-06T05:08:21.580Z caller=node_exporter.go:115 level=info collector=xfs
Dec 06 05:08:21 vagrant node_exporter[3452]: ts=2021-12-06T05:08:21.580Z caller=node_exporter.go:115 level=info collector=zfs
Dec 06 05:08:21 vagrant node_exporter[3452]: ts=2021-12-06T05:08:21.580Z caller=node_exporter.go:199 level=info msg="Listening on" address=0.0.0.0:9100
Dec 06 05:08:21 vagrant node_exporter[3452]: ts=2021-12-06T05:08:21.581Z caller=tls_config.go:195 level=info msg="TLS is disabled." http2=false
```

### 2. Ознакомьтесь с опциями node_exporter и выводом `/metrics` по-умолчанию. Приведите несколько опций, которые вы бы выбрали для базового мониторинга хоста по CPU, памяти, диску и сети.

### 3. Установите в свою виртуальную машину [Netdata](https://github.com/netdata/netdata). Воспользуйтесь [готовыми пакетами](https://packagecloud.io/netdata/netdata/install) для установки (`sudo apt install -y netdata`). После успешной установки:
    * в конфигурационном файле `/etc/netdata/netdata.conf` в секции [web] замените значение с localhost на `bind to = 0.0.0.0`,
    * добавьте в Vagrantfile проброс порта Netdata на свой локальный компьютер и сделайте `vagrant reload`:

    ```bash
    config.vm.network "forwarded_port", guest: 19999, host: 19999
    ```

   После успешной перезагрузки в браузере *на своем ПК* (не в виртуальной машине) вы должны суметь зайти на `localhost:19999`. Ознакомьтесь с метриками, которые по умолчанию собираются Netdata и с комментариями, которые даны к этим метрикам.

### 4. Можно ли по выводу `dmesg` понять, осознает ли ОС, что загружена не на настоящем оборудовании, а на системе виртуализации?

### 5. Как настроен sysctl `fs.nr_open` на системе по-умолчанию? Узнайте, что означает этот параметр. Какой другой существующий лимит не позволит достичь такого числа (`ulimit --help`)?

### 6. Запустите любой долгоживущий процесс (не `ls`, который отработает мгновенно, а, например, `sleep 1h`) в отдельном неймспейсе процессов; покажите, что ваш процесс работает под PID 1 через `nsenter`. Для простоты работайте в данном задании под root (`sudo -i`). Под обычным пользователем требуются дополнительные опции (`--map-root-user`) и т.д.

### 7. Найдите информацию о том, что такое `:(){ :|:& };:`. Запустите эту команду в своей виртуальной машине Vagrant с Ubuntu 20.04 (**это важно, поведение в других ОС не проверялось**). Некоторое время все будет "плохо", после чего (минуты) – ОС должна стабилизироваться. Вызов `dmesg` расскажет, какой механизм помог автоматической стабилизации. Как настроен этот механизм по-умолчанию, и как изменить число процессов, которое можно создать в сессии?
