# Домашнее задание к занятию "3.5. Файловые системы"

1. Узнайте о [sparse](https://ru.wikipedia.org/wiki/%D0%A0%D0%B0%D0%B7%D1%80%D0%B5%D0%B6%D1%91%D0%BD%D0%BD%D1%8B%D0%B9_%D1%84%D0%B0%D0%B9%D0%BB) (разряженных) файлах.   
    ```   
    Разреженный файл - это файл, который содержит только часть данных, которые реально используются. Когда данные записываются в разреженный файл, операционная система выделяет только действительно необходимое дисковое пространство. Таким образом, разреженный файл позволяет экономить место на диске и ускоряет операции копирования и перемещения файлов.   
    Однако, использование разреженных файлов может вызвать проблемы при переносе файлов на другие файловые системы, так как не все файловые системы поддерживают разреженные файлы или поддерживают их не полностью. Также, при использовании разреженных файлов следует учитывать, что они могут увеличить время чтения и записи данных.
    ```


1. Могут ли файлы, являющиеся жесткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?    
    ```
    Нет, файлы, являющиеся жесткой ссылкой на один объект, не могут иметь разные права доступа и владельца, потому что они ссылаются на один и тот же inode, который содержит информацию о правах доступа и владельце файла. Однако, могут быть различия в других атрибутах файла, таких как имя файла, время создания, и т.д.
    ```

1. Сделайте `vagrant destroy` на имеющийся инстанс Ubuntu. Замените содержимое Vagrantfile следующим:

    ```bash
    Vagrant.configure("2") do |config|
      config.vm.box = "bento/ubuntu-20.04"
      config.vm.provider :virtualbox do |vb|
        lvm_experiments_disk0_path = "/tmp/lvm_experiments_disk0.vmdk"
        lvm_experiments_disk1_path = "/tmp/lvm_experiments_disk1.vmdk"
        vb.customize ['createmedium', '--filename', lvm_experiments_disk0_path, '--size', 2560]
        vb.customize ['createmedium', '--filename', lvm_experiments_disk1_path, '--size', 2560]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk0_path]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk1_path]
      end
    end
    ```

    Данная конфигурация создаст новую виртуальную машину с двумя дополнительными неразмеченными дисками по 2.5 Гб.
    
        ```
        Сделано
        ```

1. Используя `fdisk`, разбейте первый диск на 2 раздела: 2 Гб, оставшееся пространство.   

    ```
    vagrant@vagrant:~$ sudo fdisk /dev/sdb

    Command (m for help): n
    Partition type
       p   primary (0 primary, 0 extended, 4 free)
       e   extended (container for logical partitions)
    Select (default p): p
    Partition number (1-4, default 1): 1
    First sector (2048-5242879, default 2048): 2048
    Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-5242879, default 5242879): +2G

    Created a new partition 1 of type 'Linux' and of size 2 GiB.

    Command (m for help): n
    Partition type
       p   primary (1 primary, 0 extended, 3 free)
       e   extended (container for logical partitions)
    Select (default p): p
    Partition number (2-4, default 2): 2
    First sector (4196352-5242879, default 4196352): 4196352
    Last sector, +/-sectors or +/-size{K,M,G,T,P} (4196352-5242879, default 5242879):

    Created a new partition 2 of type 'Linux' and of size 511 MiB.

    Command (m for help): w
    The partition table has been altered.
    Calling ioctl() to re-read partition table.
    Syncing disks.


    ```

3. Используя `sfdisk`, перенесите данную таблицу разделов на второй диск.  

    ```
    vagrant@vagrant:~$ sudo sfdisk -d /dev/sdb | sudo sfdisk /dev/sdc
    Checking that no-one is using this disk right now ... OK

    Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
    Disk model: VBOX HARDDISK
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0x563918d7

    Old situation:

    Device     Boot Start     End Sectors  Size Id Type
    /dev/sdc1        2048 5242879 5240832  2.5G 83 Linux

    >>> Script header accepted.
    >>> Script header accepted.
    >>> Script header accepted.
    >>> Script header accepted.
    >>> Created a new DOS disklabel with disk identifier 0x563918d7.
    /dev/sdc1: Created a new partition 1 of type 'Linux' and of size 2 GiB.
    /dev/sdc2: Created a new partition 2 of type 'Linux' and of size 511 MiB.
    /dev/sdc3: Done.

    New situation:
    Disklabel type: dos
    Disk identifier: 0x563918d7

    Device     Boot   Start     End Sectors  Size Id Type
    /dev/sdc1          2048 4196351 4194304    2G 83 Linux
    /dev/sdc2       4196352 5242879 1046528  511M 83 Linux

    The partition table has been altered.
    Calling ioctl() to re-read partition table.
    Syncing disks.


    ```

1. Соберите `mdadm` RAID1 на паре разделов 2 Гб.   

            ```
            vagrant@vagrant:~$ sudo mdadm --create /dev/md0 --level=mirror --raid-devices=2 /dev/sdb1 /dev/sdc1
        mdadm: Note: this array has metadata at the start and
            may not be suitable as a boot device.  If you plan to
            store '/boot' on this device please ensure that
            your boot-loader understands md/v1.x metadata, or use
            --metadata=0.90
        Continue creating array? y
        mdadm: Defaulting to version 1.2 metadata
        mdadm: array /dev/md0 started.
        vagrant@vagrant:~$ sudo mdadm --detail /dev/md0
        /dev/md0:
                   Version : 1.2
             Creation Time : Sun May  7 04:37:31 2023
                Raid Level : raid1
                Array Size : 2094080 (2045.00 MiB 2144.34 MB)
             Used Dev Size : 2094080 (2045.00 MiB 2144.34 MB)
              Raid Devices : 2
             Total Devices : 2
               Persistence : Superblock is persistent

               Update Time : Sun May  7 04:37:42 2023
                     State : clean
            Active Devices : 2
           Working Devices : 2
            Failed Devices : 0
             Spare Devices : 0

        Consistency Policy : resync

                      Name : vagrant:0  (local to host vagrant)
                      UUID : a33b2d91:a8002eff:3c24a327:34435ea4
                    Events : 17

            Number   Major   Minor   RaidDevice State
               0       8       17        0      active sync   /dev/sdb1
               1       8       33        1      active sync   /dev/sdc1

            ```

1. Соберите `mdadm` RAID0 на второй паре маленьких разделов.   
        ```   
        
            vagrant@vagrant:~$ sudo mdadm --create /dev/md1 --level=stripe --raid-devices=2 /dev/sdb2 /dev/sdc2
        mdadm: Defaulting to version 1.2 metadata
        mdadm: array /dev/md1 started.
        vagrant@vagrant:~$ sudo mdadm --detail /dev/md1
        /dev/md1:
                   Version : 1.2
             Creation Time : Sun May  7 04:42:18 2023
                Raid Level : raid0
                Array Size : 1042432 (1018.00 MiB 1067.45 MB)
              Raid Devices : 2
             Total Devices : 2
               Persistence : Superblock is persistent

               Update Time : Sun May  7 04:42:18 2023
                     State : clean
            Active Devices : 2
           Working Devices : 2
            Failed Devices : 0
             Spare Devices : 0

                    Layout : -unknown-
                Chunk Size : 512K

        Consistency Policy : none

                      Name : vagrant:1  (local to host vagrant)
                      UUID : 732daeed:97899a01:6d733f42:9d1c4d1b
                    Events : 0

            Number   Major   Minor   RaidDevice State
               0       8       18        0      active sync   /dev/sdb2
               1       8       34        1      active sync   /dev/sdc2

         ```

1. Создайте 2 независимых PV на получившихся md-устройствах.   
     ```   
     
      vagrant@vagrant:~$ sudo pvcreate /dev/md0
      Physical volume "/dev/md0" successfully created.
      vagrant@vagrant:~$ sudo pvcreate /dev/md1
      Physical volume "/dev/md1" successfully created.
      
      ```

1. Создайте общую volume-group на этих двух PV.   
    ```
     sudo vgscan
      Found volume group "ubuntu-vg" using metadata type lvm2
    vagrant@vagrant:~$ sudo vgcreate my_vg /dev/md0 /dev/md1
      Volume group "my_vg" successfully created

    ```

1. Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.   
``` 
sudo pvs
  PV         VG        Fmt  Attr PSize    PFree
  /dev/md0   my_vg     lvm2 a--    <2.00g   <2.00g
  /dev/md1   my_vg     lvm2 a--  1016.00m 1016.00m
  /dev/sda3  ubuntu-vg lvm2 a--   <63.00g  <31.50g

```

1. Создайте `mkfs.ext4` ФС на получившемся LV.

1. Смонтируйте этот раздел в любую директорию, например, `/tmp/new`.

1. Поместите туда тестовый файл, например `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.

1. Прикрепите вывод `lsblk`.

1. Протестируйте целостность файла:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```

1. Используя pvmove, переместите содержимое PV с RAID0 на RAID1.

1. Сделайте `--fail` на устройство в вашем RAID1 md.

1. Подтвердите выводом `dmesg`, что RAID1 работает в деградированном состоянии.

1. Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```

1. Погасите тестовый хост, `vagrant destroy`.

 
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
