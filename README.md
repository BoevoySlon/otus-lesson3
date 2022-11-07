# otus-lesson3

## Введение в работу с LVM

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

## Расширение LVM

Создал PV на устройстве /dev/sdc  
Расширил VG "otus" на устройство /dev/sdc  
Проверил PV в VG "otus" и его размер   
```
[vagrant@lvm ~]$ sudo pvcreate /dev/sdc
  Physical volume "/dev/sdc" successfully created.
[vagrant@lvm ~]$ sudo vgextend otus /dev/sdc
  Volume group "otus" successfully extended
[vagrant@lvm ~]$ sudo vgdisplay -v otus | grep 'PV Name'
  PV Name               /dev/sdb
  PV Name               /dev/sdc
[vagrant@lvm ~]$ sudo vgs
  VG         #PV #LV #SN Attr   VSize   VFree
  VolGroup00   1   2   0 wz--n- <38.97g     0
  otus         2   2   0 wz--n-  11.99g <3.90g
```

Симмитировал заполнение раздела данными.
```
[vagrant@lvm ~]$ sudo dd if=/dev/zero of=/data/test.log bs=1M count=8000  
status =progress
7903117312 bytes (7.9 GB) copied, 19.061696 s, 415 MB/s
dd: error writing '/data/test.log': No space left on device
7880+0 records in
7879+0 records out
8262189056 bytes (8.3 GB) copied, 19.9629 s, 414 MB/s
[vagrant@lvm ~]$ df -Th /data/
Filesystem            Type  Size  Used Avail Use% Mounted on
/dev/mapper/otus-test ext4  7.8G  7.8G     0 100% /data
```

Увеличил LV "test" до 80% от текущего объема VG "otus"
```
[vagrant@lvm ~]$ sudo lvextend -l+80%FREE /dev/otus/test
  Size of logical volume otus/test changed from <8.00 GiB (2047 extents) to <11.12 GiB (2846 extents).
  Logical volume otus/test successfully resized.
[vagrant@lvm ~]$ sudo  lvs /dev/otus/test
  LV   VG   Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  test otus -wi-ao---- <11.12g
```

Растянул файловую систему на весь объем LV "test"
```
[vagrant@lvm ~]$ df -Th /data
Filesystem            Type  Size  Used Avail Use% Mounted on
/dev/mapper/otus-test ext4  7.8G  7.8G     0 100% /data
[vagrant@lvm ~]$ sudo  resize2fs /dev/otus/test
resize2fs 1.42.9 (28-Dec-2013)
Filesystem at /dev/otus/test is mounted on /data; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 2
The filesystem on /dev/otus/test is now 2914304 blocks long.
```

## Уменьшение LV

Для уменшения раздела отмонтировал раздел от каталога, проверил на ошибки.  
Уменьшил размер файловой системы до 10Гб  
Уменьшил размер LV "test" до 10Гб  
Примонтировал раздел к каталогу  
Проверил размер раздела  
```
[vagrant@lvm ~]$ sudo umount /data/
[vagrant@lvm ~]$ sudo e2fsck -fy /dev/otus/test
e2fsck 1.42.9 (28-Dec-2013)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
/dev/otus/test: 12/729088 files (0.0% non-contiguous), 2105907/2914304 blocks
[vagrant@lvm ~]$ sudo resize2fs /dev/otus/test 10G
resize2fs 1.42.9 (28-Dec-2013)
Resizing the filesystem on /dev/otus/test to 2621440 (4k) blocks.
The filesystem on /dev/otus/test is now 2621440 blocks long.

[vagrant@lvm ~]$ sudo lvreduce /dev/otus/test -L 10G
  WARNING: Reducing active logical volume to 10.00 GiB.
  THIS MAY DESTROY YOUR DATA (filesystem etc.)
Do you really want to reduce otus/test? [y/n]: y
  Size of logical volume otus/test changed from <11.12 GiB (2846 extents) to 10.00 GiB (2560 extents).
  Logical volume otus/test successfully resized.
[vagrant@lvm ~]$ sudo  mount /dev/otus/test /data/
[vagrant@lvm ~]$ sudo df -Th /data/
Filesystem            Type  Size  Used Avail Use% Mounted on /dev/mapper/otus-test ext4  9.8G  7.8G  1.6G  84% /data
[vagrant@lvm ~]$
```

