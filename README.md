# Lesson-3-4-LVM

Уменьшить том под / до 8G.


## Выделить том под /home.

подключил к виртуальной машине диск 5Гб (ыви)
```
root@lp-ubn2:/home/sadmin# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   30G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   28G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   14G  0 lvm  /
sdb                         8:16   0    5G  0 disk
sr0                        11:0    1  2.6G  0 rom

root@lp-ubn2:/home/sadmin# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
root@lp-ubn2:/home/sadmin# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   30G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   28G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   14G  0 lvm  /
sdb                         8:16   0    5G  0 disk
sr0                        11:0    1  2.6G  0 rom
root@lp-ubn2:/home/sadmin# pvs
  PV         VG        Fmt  Attr PSize   PFree
  /dev/sda3  ubuntu-vg lvm2 a--  <28.00g 14.00g
  /dev/sdb             lvm2 ---    5.00g  5.00g

#создал группу
root@lp-ubn2:/home/sadmin# sudo vgcreate vol_gr_home /dev/sdb
  Volume group "vol_gr_home" successfully created

root@lp-ubn2:/home/sadmin# vgs
  VG          #PV #LV #SN Attr   VSize   VFree
  ubuntu-vg     1   1   0 wz--n- <28.00g 14.00g
  vol_gr_home   1   0   0 wz--n-  <5.00g <5.00g

#создал LV
root@lp-ubn2:/home/sadmin# sudo lvcreate -L 5G -n lv_home vol_gr_home
  Volume group "vol_gr_home" has insufficient free space (1279 extents): 1280 required.
root@lp-ubn2:/home/sadmin# sudo lvcreate -L 4G -n lv_home vol_gr_home
  Logical volume "lv_home" created.


root@lp-ubn2:/home/sadmin# lvs
  LV        VG          Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  ubuntu-lv ubuntu-vg   -wi-ao---- <14.00g
  lv_home   vol_gr_home -wi-a-----   4.00g

#разметил том
root@lp-ubn2:/home/sadmin# sudo mkfs.ext4 /dev/vol_gr_home/lv_home
mke2fs 1.47.0 (5-Feb-2023)
Discarding device blocks: done
Creating filesystem with 1048576 4k blocks and 262144 inodes
Filesystem UUID: e8a77746-423e-4f95-93f3-67a166714b69
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

# смонтировал папку /home
root@lp-ubn2:/home/sadmin# mount /dev/vol_gr_home/lv_home /home
root@lp-ubn2:/home/sadmin# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   30G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   28G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   14G  0 lvm  /
sdb                         8:16   0    5G  0 disk
└─vol_gr_home-lv_home     252:1    0    4G  0 lvm  /home
sr0                        11:0    1  2.6G  0 rom




```

Выделить том под /var - сделать в mirror.


Выделить том под /home - сделать том для снапшотов.


Прописать монтирование в fstab. Попробовать с разными опциями и разными файловыми системами (на выбор).
Работа со снапшотами:
сгенерить файлы в /home/;
снять снапшот;
удалить часть файлов;
восстановиться со снапшота.
