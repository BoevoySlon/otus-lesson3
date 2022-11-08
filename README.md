
На имеĀщемсā образе centos/7 - v. 1804.2  

1) Уменьшить том под / до 8G  
2) Выделить том под /var - сделать в mirror
3) Выделить том под /home      
4) /home - сделать том для снапшотов  
5) Прописать монтирование в fstab. 

Работа со снапшотами:  
- сгенерить файлы в /home/  
- снять снапшот  
- удалить часть файлов  
- восстановится со снапшота

## 1) Уменьшить том под / до 8G  

Установил xfsdump
```
[vagrant@lvm ~]$ sudo yum install xfsdump  
...
Installed:
  xfsdump.x86_64 0:3.1.7-2.el7_9

Dependency Installed:
  attr.x86_64 0:2.4.46-13.el7
```

Создал VG и LV
```
[vagrant@lvm ~]$ sudo pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
[vagrant@lvm ~]$ sudo vgcreate vg_root /dev/sdb
  Volume group "vg_root" successfully created
[vagrant@lvm ~]$ sudo  lvcreate -n lv_root -l +100%FREE /dev/vg_root
  Logical volume "lv_root" created.
```

Создал файловую систему на созданном LV и смонтировал в /mnt
```
[vagrant@lvm ~]$ sudo mkfs.xfs /dev/vg_root/lv_root
meta-data=/dev/vg_root/lv_root   isize=512    agcount=4, agsize=655104 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=2620416, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[vagrant@lvm ~]$ sudo mount /dev/vg_root/lv_root /mnt
[vagrant@lvm ~]$ df -h | grep /mnt
/dev/mapper/vg_root-lv_root       10G   33M   10G   1% /mnt
```

Сделал дамп корневой системы и восстановил его в /mnt. Поверил наличие каталогов.
```
[vagrant@lvm /]$ sudo -i
[root@lvm ~]# xfsdump -J - /dev/VolGroup00/LogVol00 | xfsrestore -J - /mnt
xfsrestore: using file dump (drive_simple) strategy
...
xfsdump: dump complete: 14 seconds elapsed
xfsdump: Dump Status: SUCCESS
xfsrestore: restore complete: 14 seconds elapsed
xfsrestore: Restore Status: SUCCESS
[vagrant@lvm /]$ cd /mnt
[vagrant@lvm mnt]$ ls
bin   dev  home  lib64  mnt  proc  run   srv  tmp  vagrant
boot  etc  lib   media  opt  root  sbin  sys  usr  var
```

Смонтировал каталоги из корня в /mnt  
Сменил корневую файловую систему на /mnt  
Сконфигурировал загрузчик  

```
[root@lvm boot]# for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
[root@lvm boot]# chroot /mnt/
[root@lvm /]#  grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-862.2.3.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img
done
```

Обновил образ initrd  

```
[root@lvm grub2]# cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g; s/.img//g"` --force; done
...
*** Store current command line parameters ***
*** Creating image file ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img
' done ***
```
Отредактировал /boot/grub2/grub.cfg, заменил rd.lvm.lv=VolGroup00/LogVol00 на rd.lvm.lv=vg_root/lv_root  
Перезагрузил ВМ
Проверил смонтированные устройства

```
[vagrant@lvm ~]$ lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk
├─sda1                    8:1    0    1M  0 part
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part
  ├─VolGroup00-LogVol00 253:1    0 37.5G  0 lvm
  └─VolGroup00-LogVol01 253:2    0  1.5G  0 lvm  [SWAP]
sdb                       8:16   0   10G  0 disk
└─vg_root-lv_root       253:0    0   10G  0 lvm  /
sdc                       8:32   0    2G  0 disk
sdd                       8:48   0    1G  0 disk
sde                       8:64   0    1G  0 disk
```

Удалил старый системный LV
Создал заново размером в 8Gb
```
[vagrant@lvm ~]$ sudo  lvremove /dev/VolGroup00/LogVol00
Do you really want to remove active logical volume VolGroup00/LogVol00? [y/n]: y

  Logical volume "LogVol00" successfully removed
[vagrant@lvm ~]$ sudo  lvcreate -n VolGroup00/LogVol00 -L 8G /dev/VolGroup00
WARNING: xfs signature detected on /dev/VolGroup00/LogVol00 at offset 0. Wipe it? [y/n]: y
  Wiping xfs signature on /dev/VolGroup00/LogVol00.
  Logical volume "LogVol00" created.
```

Создал файловую систему
Смонтировал в /mnt
Сделал дамп с корневой системы и восстановил в /mnt

```
[vagrant@lvm ~]$ sudo mkfs.xfs /dev/VolGroup00/LogVol00
meta-data=/dev/VolGroup00/LogVol00 isize=512    agcount=4, agsize=524288 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=2097152, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[vagrant@lvm ~]$ sudo mount /dev/VolGroup00/LogVol00 /mnt
[vagrant@lvm ~]$ sudo -i
[root@lvm ~]# xfsdump -J - /dev/vg_root/lv_root | xfsrestore -J - /mnt
...
xfsdump: dump complete: 14 seconds elapsed
xfsdump: Dump Status: SUCCESS
xfsrestore: restore complete: 14 seconds elapsed
xfsrestore: Restore Status: SUCCESS
```

Смонтировал каталоги из корня в /mnt  
Сменил корневую файловую систему на /mnt  
Сконфигурировал загрузчик  

```
[root@lvm ~]# for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
[root@lvm ~]# chroot /mnt/
[root@lvm /]# grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-862.2.3.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img
done
```

Обновил образ initrd  
```
[root@lvm /]# cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g; s/.img//g"` --force; done
...
*** Creating image file ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img
' done ***
```

## 2) Выделить том под /var - сделать в mirror

Создал зеркало на устройствах /dev/sdc /dev/sdd  
Создал LV размером 950Мб  

```
[root@lvm boot]# #pvcreate /dev/sdc /dev/sdd
[root@lvm boot]# pvcreate /dev/sdc /dev/sdd
  Physical volume "/dev/sdc" successfully created.
  Physical volume "/dev/sdd" successfully created.
[root@lvm boot]# vgcreate vg_var /dev/sdc /dev/sdd
  Volume group "vg_var" successfully created
[root@lvm boot]# lvcreate -L 950M -m1 -n lv_var vg_var
  Rounding up size to full physical extent 952.00 MiB
  Logical volume "lv_var" created.
 ```

Создал файловую систему  

 ```
 [root@lvm boot]# mkfs.ext4 /dev/vg_var/lv_var
 Superblock backups stored on blocks:
        32768, 98304, 163840, 229376
Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done
```

Смонтировал в /mnt
Скопировал содержимое /var/ в /mnt/  
Создал временный каталог, перенес туда содержимое /var/  

```
[root@lvm boot]# mount /dev/vg_var/lv_var /mnt
[root@lvm boot]#  cp -aR /var/* /mnt/
[root@lvm boot]#  mkdir /tmp/oldvar && mv /var/* /tmp/oldvar
```

Отмонтировал /mnt  
Смонтировал в /var  
Добавил запись fstab для автоматического монтирования при загрузке системы  

```
[root@lvm boot]# umount /mnt
[root@lvm boot]# mount /dev/vg_var/lv_var /var
[root@lvm boot]# echo "`blkid | grep var: | awk '{print $2}'` /var ext4 defaults 0 0" >> /etc/fstab
```

Перезагрузил систему  

Удалил временный LV  

```
[root@lvm ~]# vremove /dev/vg_root/lv_root
-bash: vremove: command not found
[root@lvm ~]# lvremove /dev/vg_root/lv_root
Do you really want to remove active logical volume vg_root/lv_root? [y/n]: y
  Logical volume "lv_root" successfully removed
[root@lvm ~]# vgremove /dev/vg_root
  Volume group "vg_root" successfully removed
[root@lvm ~]# pvremove /dev/sdb
  Labels on physical volume "/dev/sdb" successfully wiped.
```

## 3) Выделить том под /home  

Создал LV и создал на нем файловую систему.  
Смонтировал в /mnt  

```
[root@lvm ~]# lvcreate -n LogVol_Home -L 2G /dev/VolGroup00
  Logical volume "LogVol_Home" created.
[root@lvm ~]# mkfs.xfs /dev/VolGroup00/LogVol_Home
meta-data=/dev/VolGroup00/LogVol_Home isize=512    agcount=4, agsize=131072 blks

         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=524288, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@lvm ~]# mount /dev/VolGroup00/LogVol_Home /mnt/
```

Скопировал содержимое /home в /mnt  
Удалил содержимое /home  
Отмонтировал LV от /mnt и смонтировал в /home  
Добавил запись в fstab для монтирования раздела при загрузке системы.  

```
[root@lvm ~]# cp -aR /home/* /mnt/
[root@lvm ~]# rm -rf /home/*
[root@lvm ~]# umount /mnt
[root@lvm ~]# mount /dev/VolGroup00/LogVol_Home /home/
[root@lvm ~]# echo "`blkid | grep Home | awk '{print $2}'` /home xfs defaults 0 0" >> /etc/fstab
```

## 4) /home - сделать том для снапшотов 

Создал 20 файлов в /home  
Создал снапшот  
Удалил часть файлов  
Отмонтировал раздел  
Восстановил снапшот  
Смонтировал обратно в /home  
Проверил наличие восстановленных файлов  

```
[root@lvm ~]# touch /home/file{1..20}
[root@lvm ~]# lvcreate -L 100MB -s -n home_snap /dev/VolGroup00/LogVol_Home
  Rounding up size to full physical extent 128.00 MiB
  Logical volume "home_snap" created.
[root@lvm ~]# rm -f /home/file{11..20}
[root@lvm ~]# umount /home
[root@lvm ~]# lvconvert --merge /dev/VolGroup00/home_snap
  Merging of volume VolGroup00/home_snap started.
  VolGroup00/LogVol_Home: Merged: 100.00%
[root@lvm ~]# mount /home
[root@lvm ~]# ls /home
file1   file12  file15  file18  file20  file5  file8
file10  file13  file16  file19  file3   file6  file9
file11  file14  file17  file2   file4   file7  vagrant
```
