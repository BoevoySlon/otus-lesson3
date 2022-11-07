
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
