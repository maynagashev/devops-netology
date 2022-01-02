# Домашнее задание к занятию "3.5. Файловые системы"

### 1. Узнайте о [sparse](https://ru.wikipedia.org/wiki/%D0%A0%D0%B0%D0%B7%D1%80%D0%B5%D0%B6%D1%91%D0%BD%D0%BD%D1%8B%D0%B9_%D1%84%D0%B0%D0%B9%D0%BB) (разряженных) файлах.

Разрежённые файлы – файлы в которых последовательности нулевых байтов "сжимаются" на уровне файловой системы (информация о последовательностях не записывается на диск, а хранится в метаданных ФС).

Примеры разрежённых файлов:
- образы дисков виртуальных машин;
- резервные копий дисков и/или разделов, созданных спец. ПО;
- часто встречаются при хранении файлов БД.

### 2. Могут ли файлы, являющиеся жесткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?

Нет, жесткие ссылки не являются полноценной сущностью файловой системы, указывают на одну и ту же inode (метаданные), в которой и хранится информация о правах и владельце файла.

### 3. Сделайте `vagrant destroy` на имеющийся инстанс Ubuntu. Замените содержимое Vagrantfile следующим:

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

```bash
vagrant@vagrant:~$ lsblk
NAME                 MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                    8:0    0   64G  0 disk
├─sda1                 8:1    0  512M  0 part /boot/efi
├─sda2                 8:2    0    1K  0 part
└─sda5                 8:5    0 63.5G  0 part
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm  /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm  [SWAP]
sdb                    8:16   0  2.5G  0 disk
sdc                    8:32   0  2.5G  0 disk
```

### 4. Используя `fdisk`, разбейте первый диск на 2 раздела: 2 Гб, оставшееся пространство.

```bash
root@vagrant:~# fdisk /dev/sdb
Welcome to fdisk (util-linux 2.34).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1):
First sector (2048-5242879, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-5242879, default 5242879): +2G

Created a new partition 1 of type 'Linux' and of size 2 GiB.

Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (2-4, default 2):
First sector (4196352-5242879, default 4196352):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (4196352-5242879, default 5242879):

Created a new partition 2 of type 'Linux' and of size 511 MiB.
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

```bash
root@vagrant:~# lsblk
NAME                 MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                    8:0    0   64G  0 disk
├─sda1                 8:1    0  512M  0 part /boot/efi
├─sda2                 8:2    0    1K  0 part
└─sda5                 8:5    0 63.5G  0 part
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm  /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm  [SWAP]
sdb                    8:16   0  2.5G  0 disk
├─sdb1                 8:17   0    2G  0 part
└─sdb2                 8:18   0  511M  0 part
sdc                    8:32   0  2.5G  0 disk
```

### 5. Используя `sfdisk`, перенесите данную таблицу разделов на второй диск.

```bash
root@vagrant:~# sfdisk --dump /dev/sdb | sfdisk /dev/sdc
Checking that no-one is using this disk right now ... OK

Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Created a new DOS disklabel with disk identifier 0x133e29c0.
/dev/sdc1: Created a new partition 1 of type 'Linux' and of size 2 GiB.
/dev/sdc2: Created a new partition 2 of type 'Linux' and of size 511 MiB.
/dev/sdc3: Done.

New situation:
Disklabel type: dos
Disk identifier: 0x133e29c0

Device     Boot   Start     End Sectors  Size Id Type
/dev/sdc1          2048 4196351 4194304    2G 83 Linux
/dev/sdc2       4196352 5242879 1046528  511M 83 Linux

The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

```bash
root@vagrant:~# lsblk
NAME                 MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                    8:0    0   64G  0 disk
├─sda1                 8:1    0  512M  0 part /boot/efi
├─sda2                 8:2    0    1K  0 part
└─sda5                 8:5    0 63.5G  0 part
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm  /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm  [SWAP]
sdb                    8:16   0  2.5G  0 disk
├─sdb1                 8:17   0    2G  0 part
└─sdb2                 8:18   0  511M  0 part
sdc                    8:32   0  2.5G  0 disk
├─sdc1                 8:33   0    2G  0 part
└─sdc2                 8:34   0  511M  0 part
```

### 6. Соберите `mdadm` RAID1 на паре разделов 2 Гб.

```bash
root@vagrant:~# mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
mdadm: size set to 2094080K
Continue creating array? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
root@vagrant:~# lsblk
NAME                 MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda                    8:0    0   64G  0 disk
├─sda1                 8:1    0  512M  0 part  /boot/efi
├─sda2                 8:2    0    1K  0 part
└─sda5                 8:5    0 63.5G  0 part
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm   /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm   [SWAP]
sdb                    8:16   0  2.5G  0 disk
├─sdb1                 8:17   0    2G  0 part
│ └─md0                9:0    0    2G  0 raid1
└─sdb2                 8:18   0  511M  0 part
sdc                    8:32   0  2.5G  0 disk
├─sdc1                 8:33   0    2G  0 part
│ └─md0                9:0    0    2G  0 raid1
└─sdc2                 8:34   0  511M  0 part
```

### 7. Соберите `mdadm` RAID0 на второй паре маленьких разделов.

```bash
root@vagrant:~# mdadm --create --verbose /dev/md1 --level=0 --raid-devices=2 /dev/sdb2 /dev/sdc2
mdadm: chunk size defaults to 512K
mdadm: partition table exists on /dev/sdb2
mdadm: partition table exists on /dev/sdb2 but will be lost or
       meaningless after creating array
mdadm: partition table exists on /dev/sdc2
mdadm: partition table exists on /dev/sdc2 but will be lost or
       meaningless after creating array
Continue creating array? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md1 started.
root@vagrant:~# lsblk
NAME                 MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda                    8:0    0   64G  0 disk
├─sda1                 8:1    0  512M  0 part  /boot/efi
├─sda2                 8:2    0    1K  0 part
└─sda5                 8:5    0 63.5G  0 part
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm   /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm   [SWAP]
sdb                    8:16   0  2.5G  0 disk
├─sdb1                 8:17   0    2G  0 part
│ └─md0                9:0    0    2G  0 raid1
└─sdb2                 8:18   0  511M  0 part
  └─md1                9:1    0 1018M  0 raid0
sdc                    8:32   0  2.5G  0 disk
├─sdc1                 8:33   0    2G  0 part
│ └─md0                9:0    0    2G  0 raid1
└─sdc2                 8:34   0  511M  0 part
  └─md1                9:1    0 1018M  0 raid0
```
### 8. Создайте 2 независимых PV на получившихся md-устройствах.

```bash
root@vagrant:~# pvcreate /dev/md0 /dev/md1
  Physical volume "/dev/md0" successfully created.
  Physical volume "/dev/md1" successfully created.
```

```bash
root@vagrant:~# pvs
  PV         VG        Fmt  Attr PSize    PFree
  /dev/md0             lvm2 ---    <2.00g   <2.00g
  /dev/md1             lvm2 ---  1018.00m 1018.00m
  /dev/sda5  vgvagrant lvm2 a--   <63.50g       0
```

### 9. Создайте общую volume-group на этих двух PV.

```bash
root@vagrant:~# vgcreate vg1 /dev/md0 /dev/md1
  Volume group "vg1" successfully created
```

```bash
root@vagrant:~# vgs
  VG        #PV #LV #SN Attr   VSize   VFree
  vg1         2   0   0 wz--n-  <2.99g <2.99g
  vgvagrant   1   2   0 wz--n- <63.50g     0
```

### 10. Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.

```bash
root@vagrant:~# lvcreate --size 100M --name lv1 vg1 /dev/md1
  Logical volume "lv1" created.
```

```bash
root@vagrant:~# lvs
  LV     VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv1    vg1       -wi-a----- 100.00m
  root   vgvagrant -wi-ao---- <62.54g
  swap_1 vgvagrant -wi-ao---- 980.00m
```

### 11. Создайте `mkfs.ext4` ФС на получившемся LV.

```bash
root@vagrant:~# mkfs.ext4 /dev/vg1/lv1
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 25600 4k blocks and 25600 inodes

Allocating group tables: done
Writing inode tables: done
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done
```

```bash
root@vagrant:~# lsblk -f
NAME                 FSTYPE            LABEL     UUID                                   FSAVAIL FSUSE% MOUNTPOINT
sda
├─sda1               vfat                        7D3B-6BE4                                 511M     0% /boot/efi
├─sda2
└─sda5               LVM2_member                 Mx3LcA-uMnN-h9yB-gC2w-qm7w-skx0-OsTz9z
  ├─vgvagrant-root   ext4                        b527b79c-7f45-4e2b-a90f-1a4e9cb477c2     56.7G     2% /
  └─vgvagrant-swap_1 swap                        fad91b1f-6eed-4e4b-8dbf-913ba5bcacc7                  [SWAP]
sdb                  linux_raid_member vagrant:0 2b38fc54-98d0-0928-e35d-0c37b7dd0331
├─sdb1               linux_raid_member vagrant:0 5c2224b1-34eb-1525-67af-e9b091b2d180
│ └─md0              LVM2_member                 yieEaQ-D6op-cBfH-6SHk-c4ha-wdBQ-xDLKk3
└─sdb2               linux_raid_member vagrant:1 2f07ab3b-1c1a-96f4-8e06-894d6d6f3805
  └─md1              LVM2_member                 SlylR6-rnTw-17T7-H454-BRof-eFRy-oKD7zO
    └─vg1-lv1        ext4                        a010f107-2929-49b4-b826-1dfcb82d2396
sdc                  linux_raid_member vagrant:0 2b38fc54-98d0-0928-e35d-0c37b7dd0331
├─sdc1               linux_raid_member vagrant:0 5c2224b1-34eb-1525-67af-e9b091b2d180
│ └─md0              LVM2_member                 yieEaQ-D6op-cBfH-6SHk-c4ha-wdBQ-xDLKk3
└─sdc2               linux_raid_member vagrant:1 2f07ab3b-1c1a-96f4-8e06-894d6d6f3805
  └─md1              LVM2_member                 SlylR6-rnTw-17T7-H454-BRof-eFRy-oKD7zO
    └─vg1-lv1        ext4                        a010f107-2929-49b4-b826-1dfcb82d2396
```

### 12. Смонтируйте этот раздел в любую директорию, например, `/tmp/new`.

```bash
root@vagrant:~# mkdir /tmp/new
root@vagrant:~# mount /dev/vg1/lv1 /tmp/new/
```

```bash
root@vagrant:~# mount | grep /tmp/new
/dev/mapper/vg1-lv1 on /tmp/new type ext4 (rw,relatime,stripe=256)

Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/vg1-lv1   93M   72K   86M   1% /tmp/new
```

### 13. Поместите туда тестовый файл, например `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.
```bash
root@vagrant:~# wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz
--2022-01-02 15:58:41--  https://mirror.yandex.ru/ubuntu/ls-lR.gz
2022-01-02 15:58:49 (3.21 MB/s) - ‘/tmp/new/test.gz’ saved [21507191/21507191]
```

```bash
root@vagrant:~# ls -lah /tmp/new
total 21M
drwxr-xr-x  3 root root 4.0K Jan  2 15:58 .
drwxrwxrwt 10 root root 4.0K Jan  2 15:55 ..
drwx------  2 root root  16K Jan  2 15:51 lost+found
-rw-r--r--  1 root root  21M Jan  2 12:15 test.gz
```

### 14. Прикрепите вывод `lsblk`.

```bash
root@vagrant:~# lsblk -f
NAME                 FSTYPE            LABEL     UUID                                   FSAVAIL FSUSE% MOUNTPOINT
sda
├─sda1               vfat                        7D3B-6BE4                                 511M     0% /boot/efi
├─sda2
└─sda5               LVM2_member                 Mx3LcA-uMnN-h9yB-gC2w-qm7w-skx0-OsTz9z
  ├─vgvagrant-root   ext4                        b527b79c-7f45-4e2b-a90f-1a4e9cb477c2     56.7G     2% /
  └─vgvagrant-swap_1 swap                        fad91b1f-6eed-4e4b-8dbf-913ba5bcacc7                  [SWAP]
sdb                  linux_raid_member vagrant:0 2b38fc54-98d0-0928-e35d-0c37b7dd0331
├─sdb1               linux_raid_member vagrant:0 5c2224b1-34eb-1525-67af-e9b091b2d180
│ └─md0              LVM2_member                 yieEaQ-D6op-cBfH-6SHk-c4ha-wdBQ-xDLKk3
└─sdb2               linux_raid_member vagrant:1 2f07ab3b-1c1a-96f4-8e06-894d6d6f3805
  └─md1              LVM2_member                 SlylR6-rnTw-17T7-H454-BRof-eFRy-oKD7zO
    └─vg1-lv1        ext4                        a010f107-2929-49b4-b826-1dfcb82d2396     65.3M    22% /tmp/new
sdc                  linux_raid_member vagrant:0 2b38fc54-98d0-0928-e35d-0c37b7dd0331
├─sdc1               linux_raid_member vagrant:0 5c2224b1-34eb-1525-67af-e9b091b2d180
│ └─md0              LVM2_member                 yieEaQ-D6op-cBfH-6SHk-c4ha-wdBQ-xDLKk3
└─sdc2               linux_raid_member vagrant:1 2f07ab3b-1c1a-96f4-8e06-894d6d6f3805
  └─md1              LVM2_member                 SlylR6-rnTw-17T7-H454-BRof-eFRy-oKD7zO
    └─vg1-lv1        ext4                        a010f107-2929-49b4-b826-1dfcb82d2396     65.3M    22% /tmp/new
```

### 15. Протестируйте целостность файла:

```bash
root@vagrant:~# gzip -t /tmp/new/test.gz
root@vagrant:~# echo $?
0
```

Done. 

### 16. Используя pvmove, переместите содержимое PV с RAID0 на RAID1.

```bash
root@vagrant:~# pvmove -n lv1 /dev/md1 /dev/md0
  /dev/md1: Moved: 24.00%
  /dev/md1: Moved: 100.00%
```

### 17. Сделайте `--fail` на устройство в вашем RAID1 md.

```bash
root@vagrant:~# mdadm --fail /dev/md0 /dev/sdb1
mdadm: set /dev/sdb1 faulty in /dev/md0
```

### 18. Подтвердите выводом `dmesg`, что RAID1 работает в деградированном состоянии.

```bash
[Sun Jan  2 16:12:35 2022] md/raid1:md0: Disk failure on sdb1, disabling device.
                           md/raid1:md0: Operation continuing on 1 devices.
```

### 19. Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:

```bash
root@vagrant:~# gzip -t /tmp/new/test.gz
root@vagrant:~# echo $?
0
```

Доступен.

### 20. Погасите тестовый хост, `vagrant destroy`.

```bash
❯ vagrant destroy
    default: Are you sure you want to destroy the 'default' VM? [y/N] y
==> default: Forcing shutdown of VM...
==> default: Destroying VM and associated drives...
```
 