
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



