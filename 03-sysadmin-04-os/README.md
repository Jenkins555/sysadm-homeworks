# Домашнее задание к занятию "3.4. Операционные системы, лекция 2"

1. На лекции мы познакомились с [node_exporter](https://github.com/prometheus/node_exporter/releases). В демонстрации его исполняемый файл запускался в background. Этого достаточно для демо, но не для настоящей production-системы, где процессы должны находиться под внешним управлением. Используя знания из лекции по systemd, создайте самостоятельно простой [unit-файл](https://www.freedesktop.org/software/systemd/man/systemd.service.html) для node_exporter:

    * поместите его в автозагрузку,
    * предусмотрите возможность добавления опций к запускаемому процессу через внешний файл (посмотрите, например, на `systemctl cat cron`),
    * удостоверьтесь, что с помощью systemctl процесс корректно стартует, завершается, а после перезагрузки автоматически поднимается.
    
 # Решение 
 #### После скачивания и распаковки архива:
`$ sudo cp node_exporter /usr/local/bin/` - копируем исполняемый файл.  
`$ sudo useradd --no-create-home --shell /bin/false node_exporter` - создаём пользователя под именем node_exporter, без создания домашней директории.  
`$ sudo chown -R node_exporter:node_exporter /usr/local/bin/node_exporter` - предоставляем доступ пользователю и группе пользователей к исполняемому файлу.  
`$ vim /etc/systemd/system/node_exporter.service` - внутри деректории сервисов администратора создаём unit - файл systemd.   

      vagrant@vagrant:~$ cat /etc/systemd/system/node_exporter.service  
      [Unit]
      Description=Node exporter service # Описание unit
      After=network-online.target       # Запускать после того, как поднялась сеть.
      [Service]
      User=node_exporter                # Пользователь, от имени которого происходит запуск.
      Group=node_exporter               # Группа пользователей.
      Type=simple                       # Cлужба будет запущена незамедлительно
      ExecStart=/usr/local/bin/node_exporter # Путь к исполняемому файлу
      [Install]
      WantedBy=multi-user.target        # При запуске этого юнита будет запущен многопользовательский режим
      
      
  `$ sudo systemctl start node_exporter` - запускаем node_exporter
  
      vagrant@vagrant:~$ sudo systemctl status node_exporter  # Проверяем статус сервиса
      node_exporter.service - Node exporter service   # Название сервиса 
      Loaded: loaded (/etc/systemd/system/node_exporter.service; enabled; vendor preset: enabled)   #Загружен.Директория юнита. 
      Active: active (running) since Wed 2023-01-11 11:32:11 UTC; 20h ago  # Состояние, время запуска.
      Main PID: 50422 (node_exporter)  # Идентификатор процесса
      Tasks: 4 (limit: 1071)  
      Memory: 2.8M  
      CGroup: /system.slice/node_exporter.service  
                └─50422 /usr/local/bin/node_exporter  
      Jan 11 11:32:11 vagrant node_exporter[50422]: ts=2023-01-11T11:32:11.625Z caller=node_exporter.go:117 level=info collector=thermal_zone  #Несколько последних выводов программы.
      Jan 11 11:32:11 vagrant node_exporter[50422]: ts=2023-01-11T11:32:11.625Z caller=node_exporter.go:117 level=info collector=time

 `$ sudo systemctl enable node_exporter` - помещаем сервис в автозагрузку.  
 `$ sudo systemctl daemon-reload` - перезагрузка systemd с сохранением всех зависимостей.(получение измененных конфигураций из файловой системы и повторное создание деревьев зависимостей)
  
  `vagrant@vagrant:~$ sudo systemctl is-enabled node_exporter ` - Проверяем наличие сервиса в автозапуске.  
  `enabled`
  
  `vagrant@vagrant:~$ sudo systemctl stop node_exporter` - Принудительное завершение работы сервиса.  
  `vagrant@vagrant:~$ sudo systemctl status node_exporter    
   Active: inactive (dead) since Thu 2023-01-12 08:33:23 UTC; 8s ago
   
   `sudo systemctl restart node_exporter` - перезапускаем сервис.  
   
   >Jan 12 08:33:23 vagrant systemd[1]: Stopping Node explorter service...   
   Jan 12 08:33:23 vagrant systemd[1]: node_exporter.service: Succeeded.  
   Jan 12 08:33:23 vagrant systemd[1]: Stopped Node explorter service.     
   Jan 12 08:38:52 vagrant systemd[1]: Started Node explorter service.  
    

#### Создадим systemd таймер, для node_exporter:

`$ sudo vim /etc/systemd/system/node_exporter.timer` - создадим unit таймера.  

[Unit]  
Description=node_exporter timer  
[Timer]  
OnUnitInactiveSec=30s  # Сработает после закрытия node_exporter
Unit=node_exporter.service  
[Install]  
WantedBy=timers.target 

`$ sudo systemctl daemon-reload` - Перезагружаем systemd.  
`$ sudo systemctl start node_exporter.timer` - Запускаем unit таймера.  
`$ sudo systemctl stop node_exporter` - Завершаем работу node_exporter.service.  

 sudo journalctl -eu node_exporter` - вызываем журнал процесса.  
 
>Jan 12 10:31:33 vagrant systemd[1]: Stopping Node explorter service...  
Jan 12 10:31:33 vagrant systemd[1]: node_exporter.service: Succeeded.  
Jan 12 10:31:33 vagrant systemd[1]: Stopped Node explorter service.  
Jan 12 10:32:10 vagrant systemd[1]: Started Node explorter service.



1. Ознакомьтесь с опциями node_exporter и выводом `/metrics` по-умолчанию. Приведите несколько опций, которые вы бы выбрали для базового мониторинга хоста по CPU, памяти, диску и сети.  
  
  # Решение:  
  Для настройки метрики в node_exporter существуют коллекторы. Флаги вида --collector.<name>. Флаги используются при вызове сервиса, так же их можно включить в unit-файл node_exporter.service.  
   
 ```
[Unit]      
Description=Node explorter service    
After=network-online.target      
[Service]      
User=node_exporter    
Group=node_exporter       
Type=simple      
ExecStart=/usr/local/bin/node_exporter --collector.disable-defaults --collector.cpu --collector.meminfo --collector.filesystem --collector.netdev  # Убираем метрики по умолчанию. Добавляем метрики CPU, памяти, диска и сети.  
[Install]    
WantedBy=multi-user.target 
   ```
   
   
 >Jan 12 13:05:31 vagrant node_exporter[54750]: ts=2023-01-12T13:05:31.886Z caller=node_exporter.go:110 level=info msg="Enabled collectors"  
   Jan 12 13:05:31 vagrant node_exporter[54750]: ts=2023-01-12T13:05:31.886Z caller=node_exporter.go:117 level=info collector=cpu  
   Jan 12 13:05:31 vagrant node_exporter[54750]: ts=2023-01-12T13:05:31.886Z caller=node_exporter.go:117 level=info collector=filesystem  
   Jan 12 13:05:31 vagrant node_exporter[54750]: ts=2023-01-12T13:05:31.886Z caller=node_exporter.go:117 level=info collector=meminfo  
   Jan 12 13:05:31 vagrant node_exporter[54750]: ts=2023-01-12T13:05:31.886Z caller=node_exporter.go:117 level=info collector=netdev  
  

   
## --collector.cpu:  
   ### Расчёт процента использования процессора.
   >node_cpu_seconds_total{cpu="0",mode="idle"} 59702.01  
   node_cpu_seconds_total{cpu="0",mode="iowait"} 369.43  
   node_cpu_seconds_total{cpu="0",mode="irq"} 0  
   node_cpu_seconds_total{cpu="0",mode="nice"} 0.16  
   node_cpu_seconds_total{cpu="0",mode="softirq"} 25.19  
   node_cpu_seconds_total{cpu="0",mode="steal"} 0  
   node_cpu_seconds_total{cpu="0",mode="system"} 1065.8  
   node_cpu_seconds_total{cpu="0",mode="user"} 538.1  
   

## --collector.meminfo:
   ### Всё что связано с памятью.   
   >HELP node_memory_MemFree_bytes Memory information field MemFree_bytes.  
    # TYPE node_memory_MemFree_bytes gauge  
    node_memory_MemFree_bytes 1.60579584e+08  
    # HELP node_memory_MemTotal_bytes Memory information field MemTotal_bytes.  
    # TYPE node_memory_MemTotal_bytes gauge  
    node_memory_MemTotal_bytes 1.028685824e+09  
   
   
## --collector.filesystem:
   ### Контроль дискового пространства.
   >node_filesystem_free_bytes{device="/dev/sda2",fstype="ext4",mountpoint="/boot"} 8.80697344e+08
    node_filesystem_avail_bytes{device="/dev/sda2",fstype="ext4",mountpoint="/boot"} 8.10233856e+08

## --collector.netstat:
   ### Метрики сети.
    ># HELP node_network_receive_bytes_total Network device statistic receive_bytes.
     # TYPE node_network_receive_bytes_total counter
     node_network_receive_bytes_total{device="eth0"} 4.5052648e+08
     # HELP node_network_transmit_bytes_total Network device statistic transmit_bytes.
     # TYPE node_network_transmit_bytes_total counter
     node_network_transmit_bytes_total{device="eth0"} 9.661276e+06
     node_network_transmit_bytes_total{device="lo"} 470443





3. Установите в свою виртуальную машину [Netdata](https://github.com/netdata/netdata). Воспользуйтесь [готовыми пакетами](https://packagecloud.io/netdata/netdata/install) для установки (`sudo apt install -y netdata`). После успешной установки:
    * в конфигурационном файле `/etc/netdata/netdata.conf` в секции [web] замените значение с localhost на `bind to = 0.0.0.0`,
    * добавьте в Vagrantfile проброс порта Netdata на свой локальный компьютер и сделайте `vagrant reload`:

    ```bash
    config.vm.network "forwarded_port", guest: 19999, host: 19999
    ```

    После успешной перезагрузки в браузере *на своем ПК* (не в виртуальной машине) вы должны суметь зайти на `localhost:19999`. Ознакомьтесь с метриками, которые по умолчанию собираются Netdata и с комментариями, которые даны к этим метрикам.

1. Можно ли по выводу `dmesg` понять, осознает ли ОС, что загружена не на настоящем оборудовании, а на системе виртуализации?
1. Как настроен sysctl `fs.nr_open` на системе по-умолчанию? Узнайте, что означает этот параметр. Какой другой существующий лимит не позволит достичь такого числа (`ulimit --help`)?
1. Запустите любой долгоживущий процесс (не `ls`, который отработает мгновенно, а, например, `sleep 1h`) в отдельном неймспейсе процессов; покажите, что ваш процесс работает под PID 1 через `nsenter`. Для простоты работайте в данном задании под root (`sudo -i`). Под обычным пользователем требуются дополнительные опции (`--map-root-user`) и т.д.
1. Найдите информацию о том, что такое `:(){ :|:& };:`. Запустите эту команду в своей виртуальной машине Vagrant с Ubuntu 20.04 (**это важно, поведение в других ОС не проверялось**). Некоторое время все будет "плохо", после чего (минуты) – ОС должна стабилизироваться. Вызов `dmesg` расскажет, какой механизм помог автоматической стабилизации. Как настроен этот механизм по-умолчанию, и как изменить число процессов, которое можно создать в сессии?

 
 ---

## Как сдавать задания

Обязательными к выполнению являются задачи без указания звездочки. Их выполнение необходимо для получения зачета и диплома о профессиональной переподготовке.

Задачи со звездочкой (*) являются дополнительными задачами и/или задачами повышенной сложности. Они не являются обязательными к выполнению, но помогут вам глубже понять тему.

Домашнее задание выполните в файле readme.md в github репозитории. В личном кабинете отправьте на проверку ссылку на .md-файл в вашем репозитории.

Также вы можете выполнить задание в [Google Docs](https://docs.google.com/document/u/0/?tgif=d) и отправить в личном кабинете на проверку ссылку на ваш документ.
Название файла Google Docs должно содержать номер лекции и фамилию студента. Пример названия: "1.1. Введение в DevOps — Сусанна Алиева".

Если необходимо прикрепить дополнительные ссылки, просто добавьте их в свой Google Docs.

Перед тем как выслать ссылку, убедитесь, что ее содержимое не является приватным (открыто на комментирование всем, у кого есть ссылка), иначе преподаватель не сможет проверить работу. Чтобы это проверить, откройте ссылку в браузере в режиме инкогнито.

[Как предоставить доступ к файлам и папкам на Google Диске](https://support.google.com/docs/answer/2494822?hl=ru&co=GENIE.Platform%3DDesktop)

[Как запустить chrome в режиме инкогнито ](https://support.google.com/chrome/answer/95464?co=GENIE.Platform%3DDesktop&hl=ru)

[Как запустить  Safari в режиме инкогнито ](https://support.apple.com/ru-ru/guide/safari/ibrw1069/mac)

Любые вопросы по решению задач задавайте в чате Slack.

---
