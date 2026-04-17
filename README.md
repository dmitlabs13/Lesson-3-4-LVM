# Lesson-3-4-LVM

## задание
На виртуальной машине с Ubuntu 24.04 и LVM.

1. Уменьшить том под / до 8G.
2. Выделить том под /home.
3. Выделить том под /var - сделать в mirror.
4. /home - сделать том для снапшотов.
5. Прописать монтирование в fstab. Попробовать с разными опциями и разными файловыми системами (на выбор).
6. Работа со снапшотами:
6.1. сгенерить файлы в /home/;
6.2. снять снапшот;
6.3. удалить часть файлов;
6.4. восстановиться со снапшота




## 1. Уменьшить том под / до 8G.


## 2. Выделить том под /home.


```
# подключил к виртуальной машине диск 20Гб (sdb)

root@lp-ubn2:/home/sadmin# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   30G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   28G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   14G  0 lvm  /
sdb                         8:16   0   20G  0 disk
sr0                        11:0    1  2.6G  0 rom

#Создал физический том
root@lp-ubn2:/home/sadmin# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.


# вижу новый PV
root@lp-ubn2:/home/sadmin# pvs
  PV         VG        Fmt  Attr PSize   PFree
  /dev/sda3  ubuntu-vg lvm2 a--  <28.00g 14.00g
  /dev/sdb             lvm2 ---   20.00g 20.00g


#создал группу vgroupe_home 
root@lp-ubn2:/home/sadmin# vgcreate vgroupe_home /dev/sdb
  Volume group "vgroupe_home" successfully created
root@lp-ubn2:/home/sadmin# vgs
  VG           #PV #LV #SN Attr   VSize   VFree
  ubuntu-vg      1   1   0 wz--n- <28.00g  14.00g
  vgroupe_home   1   0   0 wz--n- <20.00g <20.00g


#создал LV "lvolume_home"
root@lp-ubn2:/home/sadmin# lvcreate -L 10G -n lvolume_home vggroupe_home
  Volume group "vggroupe_home" not found
  Cannot process volume group vggroupe_home
root@lp-ubn2:/home/sadmin# lvcreate -L 10G -n lvolume_home vgroupe_home
  Logical volume "lvolume_home" created.
root@lp-ubn2:/home/sadmin# lvs
  LV           VG           Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  ubuntu-lv    ubuntu-vg    -wi-ao---- <14.00g
  lvolume_home vgroupe_home -wi-a-----  10.00g


#разметил том
root@lp-ubn2:/home/sadmin# mkfs.ext4 /dev/mapper/vgroupe_home-lvolume_home
mke2fs 1.47.0 (5-Feb-2023)
Discarding device blocks: done
Creating filesystem with 2621440 4k blocks and 655360 inodes
Filesystem UUID: 6f7e7547-3aed-4127-9a18-ef6391ff4acc
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done


# смонтировал папку /home
root@lp-ubn2:/home/sadmin# mkdir /home-temp
root@lp-ubn2:/home/sadmin# cp -r /home/. /home-temp
root@lp-ubn2:/home/sadmin# ls /home-temp/
sadmin
root@lp-ubn2:/home/sadmin# mount /dev/mapper/vgroupe_home-lvolume_home /home
root@lp-ubn2:/home/sadmin# ls /home
lost+found
root@lp-ubn2:/home/sadmin# cp -r /home-temp/. /home
root@lp-ubn2:/home/sadmin# ls /home
root@lp-ubn2:/home/sadmin# lsblk
NAME                        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                           8:0    0   30G  0 disk
├─sda1                        8:1    0    1M  0 part
├─sda2                        8:2    0    2G  0 part /boot
└─sda3                        8:3    0   28G  0 part
  └─ubuntu--vg-ubuntu--lv   252:0    0   14G  0 lvm  /
sdb                           8:16   0   20G  0 disk
└─vgroupe_home-lvolume_home 252:1    0   10G  0 lvm  /home


```

## 3. Выделить том под /var - сделать в mirror.


## 4. /home - сделать том для снапшотов.
```
root@lp-ubn2:/home/sadmin# lvcreate -L 3G -s -n home-snap /dev/mapper/vgroupe_home-lvolume_home
  Logical volume "home-snap" created.


```

## 5. Прописать монтирование в fstab. Попробовать с разными опциями и разными файловыми системами (на выбор).
```
# Смотрим что есть сейчас и добавляем нашу строку /dev/mapper/vgroupe_home-lvolume_home   /home   ext4 defaults   0 0

root@lp-ubn2:/home/sadmin# cat /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/ubuntu-vg/ubuntu-lv during curtin installation
/dev/disk/by-id/dm-uuid-LVM-4YQFaHDOjSzgMADcu7vljzMKX3kROyG8mrUDWSXvTdYtG2OcOaerfd4YWdYSIhg7 / ext4 defaults 0 1
# /boot was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/a86353c7-152f-465b-ada8-2c190269fb5e /boot ext4 defaults 0 1
/swap.img       none    swap    sw      0       0
/dev/mapper/vgroupe_home-lvolume_home   /home   ext4 defaults   0 0

#проверяем
root@lp-ubn2:/home/sadmin# systemctl daemon-reload
root@lp-ubn2:/home/sadmin# umount /home
root@lp-ubn2:/home/sadmin# mount /home
root@lp-ubn2:/home/sadmin# lsblk
NAME                             MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                                8:0    0   30G  0 disk
├─sda1                             8:1    0    1M  0 part
├─sda2                             8:2    0    2G  0 part /boot
└─sda3                             8:3    0   28G  0 part
  └─ubuntu--vg-ubuntu--lv        252:0    0   14G  0 lvm  /
sdb                                8:16   0   20G  0 disk
├─vgroupe_home-lvolume_home-real 252:2    0   10G  0 lvm
│ ├─vgroupe_home-lvolume_home    252:1    0   10G  0 lvm  /home
│ └─vgroupe_home-home--snap      252:4    0   10G  0 lvm
└─vgroupe_home-home--snap-cow    252:3    0    3G  0 lvm
  └─vgroupe_home-home--snap      252:4    0   10G  0 lvm
sr0                               11:0    1  2.6G  0 rom




```
## 6. Работа со снапшотами:
## 6.1. сгенерить файлы в /home/
```
root@lp-ubn2:/home/sadmin# cd /home
root@lp-ubn2:/home# touch file{1..30}.{txt,log,csv,json}
root@lp-ubn2:/home# ls
file10.csv   file12.txt   file15.log   file18.json  file20.csv   file22.txt   file25.log   file28.json  file30.csv   file4.txt   file7.log   sadmin
file10.json  file13.csv   file15.txt   file18.log   file20.json  file23.csv   file25.txt   file28.log   file30.json  file5.csv   file7.txt
file10.log   file13.json  file16.csv   file18.txt   file20.log   file23.json  file26.csv   file28.txt   file30.log   file5.json  file8.csv
file10.txt   file13.log   file16.json  file19.csv   file20.txt   file23.log   file26.json  file29.csv   file30.txt   file5.log   file8.json
file11.csv   file13.txt   file16.log   file19.json  file21.csv   file23.txt   file26.log   file29.json  file3.csv    file5.txt   file8.log
file11.json  file14.csv   file16.txt   file19.log   file21.json  file24.csv   file26.txt   file29.log   file3.json   file6.csv   file8.txt
file11.log   file14.json  file17.csv   file19.txt   file21.log   file24.json  file27.csv   file29.txt   file3.log    file6.json  file9.csv
file11.txt   file14.log   file17.json  file1.csv    file21.txt   file24.log   file27.json  file2.csv    file3.txt    file6.log   file9.json
file12.csv   file14.txt   file17.log   file1.json   file22.csv   file24.txt   file27.log   file2.json   file4.csv    file6.txt   file9.log
file12.json  file15.csv   file17.txt   file1.log    file22.json  file25.csv   file27.txt   file2.log    file4.json   file7.csv   file9.txt
file12.log   file15.json  file18.csv   file1.txt    file22.log   file25.json  file28.csv   file2.txt    file4.log    file7.json  lost+found

```
## 6.2. снять снапшот


## 6.3. удалить часть файлов


## 6.4. восстановиться со снапшота
