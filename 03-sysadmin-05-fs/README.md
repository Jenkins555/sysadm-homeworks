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


     vagrant@vagrant:~$ sudo lvcreate -L 100M -n my_lv my_vg /dev/md1
      Logical volume "my_lv" created.   


      vagrant@vagrant:~$ sudo lvdisplay
         --- Logical volume ---
      LV Path                /dev/my_vg/my_lv
      LV Name                my_lv
      VG Name                my_vg
      LV UUID                Exuol0-Oz7o-Ck9R-4dbd-SX7y-M01h-Q5HEof
      LV Write Access        read/write
      LV Creation host, time vagrant, 2023-05-08 04:10:12 +0000
      LV Status              available
      # open                 0
      LV Size                100.00 MiB
      Current LE             25
      Segments               1
      Allocation             inherit
      Read ahead sectors     auto
      - currently set to     4096
      Block device           253:1


    ```

1. Создайте `mkfs.ext4` ФС на получившемся LV.   
      ```
      vagrant@vagrant:~$ sudo mkfs.ext4 /dev/my_vg/my_lv
    mke2fs 1.45.5 (07-Jan-2020)
    Creating filesystem with 25600 4k blocks and 25600 inodes

    Allocating group tables: done
    Writing inode tables: done
    Creating journal (1024 blocks): done
    Writing superblocks and filesystem accounting information: done

      ```

1. Смонтируйте этот раздел в любую директорию, например, `/tmp/new`.    
  
    ```
    vagrant@vagrant:~$ sudo mkdir /tmp/new
    vagrant@vagrant:~$ sudo mount /dev/my_vg/my_lv /tmp/new
    vagrant@vagrant:~$ mount | grep /tmp/new
    /dev/mapper/my_vg-my_lv on /tmp/new type ext4 (rw,relatime,stripe=256)

    ```

1. Поместите туда тестовый файл, например `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.   

    ```
     vagrant@vagrant:~$ ls /tmp/new
     lost+found  test.gz

    ```

1. Прикрепите вывод `lsblk`.   
     ```
         vagrant@vagrant:~$ lsblk
        NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
        loop0                       7:0    0 55.6M  1 loop  /snap/core18/2721
        loop1                       7:1    0 63.3M  1 loop  /snap/core20/1879
        loop2                       7:2    0 55.4M  1 loop  /snap/core18/2128
        loop3                       7:3    0 70.3M  1 loop  /snap/lxd/21029
        loop4                       7:4    0 91.9M  1 loop  /snap/lxd/24061
        loop5                       7:5    0 32.3M  1 loop  /snap/snapd/12704
        sda                         8:0    0   64G  0 disk
        ├─sda1                      8:1    0    1M  0 part
        ├─sda2                      8:2    0    1G  0 part  /boot
        └─sda3                      8:3    0   63G  0 part
          └─ubuntu--vg-ubuntu--lv 253:0    0 31.5G  0 lvm   /
        sdb                         8:16   0  2.5G  0 disk
        ├─sdb1                      8:17   0    2G  0 part
        │ └─md0                     9:0    0    2G  0 raid1
        └─sdb2                      8:18   0  511M  0 part
          └─md1                     9:1    0 1018M  0 raid0
            └─my_vg-my_lv         253:1    0  100M  0 lvm   /tmp/new
        sdc                         8:32   0  2.5G  0 disk
        ├─sdc1                      8:33   0    2G  0 part
        │ └─md0                     9:0    0    2G  0 raid1
        └─sdc2                      8:34   0  511M  0 part
          └─md1                     9:1    0 1018M  0 raid0
            └─my_vg-my_lv         253:1    0  100M  0 lvm   /tmp/new

     ```

1. Протестируйте целостность файла:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```  
    
    ```
    vagrant@vagrant:~$ gzip -t /tmp/new/test.gz
    vagrant@vagrant:~$ echo $?
    0
  
    ```

1. Используя pvmove, переместите содержимое PV с RAID0 на RAID1.   

      ```
      vagrant@vagrant:~$ sudo pvs
      PV         VG        Fmt  Attr PSize    PFree
      /dev/md0   my_vg     lvm2 a--    <2.00g  <2.00g
      /dev/md1   my_vg     lvm2 a--  1016.00m 916.00m
      /dev/sda3  ubuntu-vg lvm2 a--   <63.00g <31.50g
    vagrant@vagrant:~$ sudo pvmove /dev/md1 /dev/md0
      /dev/md1: Moved: 12.00%
      /dev/md1: Moved: 100.00%
    vagrant@vagrant:~$ sudo pvs
      PV         VG        Fmt  Attr PSize    PFree
      /dev/md0   my_vg     lvm2 a--    <2.00g   <1.90g
      /dev/md1   my_vg     lvm2 a--  1016.00m 1016.00m
      /dev/sda3  ubuntu-vg lvm2 a--   <63.00g  <31.50g

      ```

1. Сделайте `--fail` на устройство в вашем RAID1 md.   
    ```
        vagrant@vagrant:~$ sudo mdadm --manage /dev/md0 --fail /dev/sdb1
        mdadm: set /dev/sdb1 faulty in /dev/md0

    

    ```

1. Подтвердите выводом `dmesg`, что RAID1 работает в деградированном состоянии.

      ```
          vagrant@vagrant:~$ dmesg | grep md0
        [    2.703773] md/raid1:md0: active with 2 out of 2 mirrors
        [    2.704806] md0: detected capacity change from 0 to 2144337920
        [ 2056.125897] md: data-check of RAID array md0
        [ 2066.858362] md: md0: data-check done.
        [ 9743.741749] md/raid1:md0: Disk failure on sdb1, disabling device.
                       md/raid1:md0: Operation continuing on 1 devices.

      ```

1. Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```
    
    ```
    vagrant@vagrant:~$ gzip -t /tmp/new/test.gz
    vagrant@vagrant:~$ echo $?
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
