
```
[vagrant@lvm ~]$ sudo yum install xfsdump  
...
Installed:
  xfsdump.x86_64 0:3.1.7-2.el7_9

Dependency Installed:
  attr.x86_64 0:2.4.46-13.el7
```


```
[vagrant@lvm ~]$ sudo pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
[vagrant@lvm ~]$ sudo vgcreate vg_root /dev/sdb
  Volume group "vg_root" successfully created
[vagrant@lvm ~]$ sudo  lvcreate -n lv_root -l +100%FREE /dev/vg_root
  Logical volume "lv_root" created.
```

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

```
[vagrant@lvm /]$ sudo -i
[root@lvm ~]# xfsdump -J - /dev/VolGroup00/LogVol00 | xfsrestore -J - /mnt
...
xfsrestore: restore complete: 14 seconds elapsed
xfsrestore: Restore Status: SUCCESS
```

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

```
[root@lvm grub2]# cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g; s/.img//g"` --force; done
...
*** Store current command line parameters ***
*** Creating image file ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img
' done ***
```

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









