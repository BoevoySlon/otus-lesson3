# otus-lesson3

##Введение в работу с LVM

Развернул и запустил виртуальную машину lvm.

Проверил наличие дисков в системе
```
[vagrant@lvm ~]$ lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk
├─sda1                    8:1    0    1M  0 part
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part
  ├─VolGroup00-LogVol00 253:0    0 37.5G  0 lvm  /
  └─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
sdb                       8:16   0   10G  0 disk
sdc                       8:32   0    2G  0 disk
sdd                       8:48   0    1G  0 disk
sde                       8:64   0    1G  0 disk
```
Создал Physical volume на устройстве /dev/sdb 
```
[vagrant@lvm ~]$ sudo pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
```

Создал Volume group "otus"
```
[vagrant@lvm ~]$ sudo vgcreate otus /dev/sdb
  Volume group "otus" successfully created
```
Создал Logical volume "test" на 80% от объема VG
```
[vagrant@lvm ~]$ sudo lvcreate -l+80%FREE -n test otus
  Logical volume "test" created.
```

Посмотрел информацию о созданном VG
```
[vagrant@lvm ~]$ sudo vgdisplay otus
  --- Volume group ---
  VG Name               otus
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <10.00 GiB
  PE Size               4.00 MiB
  Total PE              2559
  Alloc PE / Size       2047 / <8.00 GiB
  Free  PE / Size       512 / 2.00 GiB
  VG UUID               fSgkKU-wFoh-Bvpa-taA8-JZzg-Jdbl-fe2Eid
```

Создал Logical volume "small" размером 100Мб
```
[vagrant@lvm ~]$ sudo lvcreate -L100M -n small otus
  Logical volume "small" created.
```

Посмотрел информацию о текущих LV
```
[vagrant@lvm ~]$ sudo lvs
  LV       VG         Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy% Sync Convert
  LogVol00 VolGroup00 -wi-ao---- <37.47g

  LogVol01 VolGroup00 -wi-ao----   1.50g

  small    otus       -wi-a----- 100.00m

  test     otus       -wi-a-----  <8.00g
```

Создал файловую систему на LV "test" и смонтировал ее в каталог /data
```
[vagrant@lvm ~]$ sudo mkfs.ext4 /dev/otus/test
...
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
[vagrant@lvm ~]$ mkdir /data
mkdir: cannot create directory '/data': Permission denied
[vagrant@lvm ~]$ sudo mkdir /data
[vagrant@lvm ~]$ sudo mount /dev/otus/test /data/
[vagrant@lvm ~]$ sudo  mount | grep /data
/dev/mapper/otus-test on /data type ext4 (rw,relatime,seclabel,data=ordered)
```

##Расширение LVM

Создал PV на устройстве /dev/sdc  
Расширил VG "otus" на устройство /dev/sdc  
Проверил PV в МП "otus"  
```
[vagrant@lvm ~]$ sudo pvcreate /dev/sdc
  Physical volume "/dev/sdc" successfully created.
[vagrant@lvm ~]$ sudo vgextend otus /dev/sdc
  Volume group "otus" successfully extended
[vagrant@lvm ~]$ sudo vgdisplay -v otus | grep 'PV Name'
  PV Name               /dev/sdb
  PV Name               /dev/sdc
```

