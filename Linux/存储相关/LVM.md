#### 说明
```txt
1. PE：     '8e'默认块:4M
2. PV：     物理卷
3. VG：     卷组
4. LV：     逻辑卷

物理卷             卷组          逻辑卷
扫描： pvscan      vgscan        lvscan
建立： pvcreate    vgcreate      lvcreate
显示： pvdisplay   vgdisplay     lvdisplay
删除： pvremove    vgremove      lvremove
扩展： ---------   vgextend      lvextend
减小： ---------   vgreduce      lvreduce
其他： pvmove      vgchange      resize2fs
```

#### 创建LV
```bash
[root@localhost ~]# lsblk | grep "^sd*"
sda             8:0    0   24G  0 disk 
sdb             8:16   0    1G  0 disk 
sdc             8:32   0    1G  0 disk 
sdd             8:48   0    1G  0 disk 
sr0            11:0    1 1024M  0 rom
[root@localhost ~]# pvcreate /dev/sd{b,c,d}             #主板新加入的磁盘设备 /dev/sd{b,c,d} 转为PV
  Physical volume "/dev/sdb" successfully created.
  Physical volume "/dev/sdc" successfully created.
  Physical volume "/dev/sdd" successfully created.
[root@localhost ~]# vgcreate -s 1M vg01 /dev/sd{b,c,d}  #指定PE大小为1MB
  Volume group "vg01" successfully created
[root@localhost ~]# vgscan 
  Reading volume groups from cache.
  Found volume group "centos" using metadata type lvm2
  Found volume group "vg01" using metadata type lvm2
[root@localhost ~]# vgdisplay vg01
  --- Volume group ---
  VG Name               vg01
  System ID             
  Format                lvm2
  Metadata Areas        3
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                3
  Act PV                3
  VG Size               3.00 GiB
  PE Size               1.00 MiB
  Total PE              3069
  Alloc PE / Size       0 / 0   
  Free  PE / Size       3069 / 3.00 GiB
  VG UUID               7VDNWl-uIUy-vEaN-lvQV-Y2we-nqnx-X83pDJ
[root@localhost ~]# lvcreate -L 1G -n lv01 vg01         #从卷组vg01划分出逻辑卷lv01并指定其固定的容量
  Logical volume "lv01" created.
[root@localhost ~]# lvdisplay lv01
  Volume group "lv01" not found
  Cannot process volume group lv01
[root@localhost ~]# lvdisplay /dev/vg01/lv01
  --- Logical volume ---
  LV Path                /dev/vg01/lv01
  LV Name                lv01
  VG Name                vg01
  LV UUID                SANZ5r-OgqO-mHYo-k9Od-UAwf-Pb0x-nKxyCO
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2017-11-10 15:56:22 +0800
  LV Status              available
  # open                 0
  LV Size                1.00 GiB
  Current LE             1024
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:3       
[root@localhost ~]# mke2fs -t ext4 -L test-volume /dev/vg01/lv01    #格式化后可挂载使用
mke2fs 1.42.9 (28-Dec-2013)
文件系统标签=test-volume
OS type: Linux
块大小=4096 (log=2)
分块大小=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
65536 inodes, 262144 blocks
13107 blocks (5.00%) reserved for the super user
第一个数据块=0
Maximum filesystem blocks=268435456
8 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376

Allocating group tables: 完成                            
正在写入inode表: 完成                            
Creating journal (8192 blocks): 完成
Writing superblocks and filesystem accounting information: 完成
[root@localhost mnt]# mount -L test-volume /mnt         #挂载
```
#### 扩容LV
```bash
[root@localhost mnt]# pvscan                            #查看剩余可用PV
  PV /dev/sda2   VG centos          lvm2 [23.51 GiB / 44.00 MiB free]
  PV /dev/sdb    VG vg01            lvm2 [1023.00 MiB / 0    free]
  PV /dev/sdc    VG vg01            lvm2 [1023.00 MiB / 1022.00 MiB free]
  PV /dev/sdd    VG vg01            lvm2 [1023.00 MiB / 1023.00 MiB free]
  Total: 4 [26.50 GiB] / in use: 4 [26.50 GiB] / in no VG: 0 [0   ]
[root@localhost mnt]# vgextend vg01 /dev/sdd            #将新增PV加入已经存在的VG（此处提示已经加入）
  Physical volume '/dev/sdd' is already in volume group 'vg01'
  Unable to add physical volume '/dev/sdd' to volume group 'vg01'
[root@localhost mnt]# lvextend -L +1G /dev/vg01/lv01    #不影响原数据（-L #G：指定扩容到#G / -L +#G： 增加#G）
  Size of logical volume vg01/lv01 changed from 1.00 GiB (1024 extents) to 2.00 GiB (2048 extents).
  Logical volume vg01/lv01 successfully resized.
[root@localhost mnt]# lvdisplay /dev/vg01/lv01
  --- Logical volume ---
  LV Path                /dev/vg01/lv01
  LV Name                lv01
  VG Name                vg01
  LV UUID                SANZ5r-OgqO-mHYo-k9Od-UAwf-Pb0x-nKxyCO
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2017-11-10 15:56:22 +0800
  LV Status              available
  # open                 1
  LV Size                2.00 GiB
  Current LE             2048
  Segments               3
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:3
[root@localhost mnt]# resize2fs /dev/vg01/lv01          #重读FS大小（默认为LV最大容量、指定：resize2fs /dev/vg01/lv01 5G）
resize2fs 1.42.9 (28-Dec-2013)
Filesystem at /dev/vg01/lv01 is mounted on /mnt; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 1
The filesystem on /dev/vg01/lv01 is now 524288 blocks long.
[root@localhost mnt]# df -aTh | grep "lv01"             #检查
/dev/mapper/vg01-lv01   ext4         2.0G  3.0M  1.9G    1% /mnt
```
#### 缩小LV
```bash
#注：除BTRFS以外的文件系统默认不支持在线缩小FS容量，参考：http://os.51cto.com/art/201310/413668.htm
[root@localhost /]# umount /dev/vg01/lv01
[root@localhost /]# e2fsck -f /dev/vg01/lv01            #执行FS检查
e2fsck 1.42.9 (28-Dec-2013)
第一步: 检查inode,块,和大小
第二步: 检查目录结构
第3步: 检查目录连接性
Pass 4: Checking reference counts
第5步: 检查簇概要信息
test-volume: 11/131072 files (0.0% non-contiguous), 17196/524288 blocks
[root@localhost /]# resize2fs /dev/vg01/lv01  1G        #减小LV容量到1G
resize2fs 1.42.9 (28-Dec-2013)
Resizing the filesystem on /dev/vg01/lv01 to 262144 (4k) blocks.
The filesystem on /dev/vg01/lv01 is now 262144 blocks long.
[root@localhost /]# lvchange -an /dev/vg01/lv01         #使LV的访问禁止
[root@localhost /]# lvreduce -L 1G  /dev/vg01/lv01      #把LV减1G容量（还有lvresize可修改其逻辑边界）  
  Size of logical volume vg01/lv01 changed from 2.00 GiB (2048 extents) to 1.00 GiB (1024 extents).
  Logical volume vg01/lv01 successfully resized.
[root@localhost /]# lvchange -ay /dev/vg01/lv01         #使LV可允许访问
[root@localhost /]# lvscan  | grep lv01
  ACTIVE            '/dev/vg01/lv01' [1.00 GiB] inherit
[root@localhost /]# e2fsck -f /dev/vg01/lv01
e2fsck 1.42.9 (28-Dec-2013)
第一步: 检查inode,块,和大小
第二步: 检查目录结构
第3步: 检查目录连接性
Pass 4: Checking reference counts
第5步: 检查簇概要信息
test-volume: 11/65536 files (0.0% non-contiguous), 12955/262144 blocks
[root@localhost /]# mount -L test-volume /mnt

#将无用的PV从VG删除：
[root@localhost /]# pvdisplay -m   
  --- Physical volume ---
  PV Name               /dev/sdb
  VG Name               vg01
  PV Size               1.00 GiB / not usable 0   
  Allocatable           yes (but full)  #提示已经在被使用中并且全部被使用
  PE Size               1.00 MiB
  Total PE              1023
  Free PE               0
  Allocated PE          1023
  PV UUID               bACZUQ-fSh3-UtGv-xcFQ-VLfr-GWcX-9wzBMd
   
  --- Physical Segments ---   #PE /dev/sdb 所在的LV
  Physical extent 0 to 1022:
    Logical volume      /dev/vg01/lv01
    Logical extents     0 to 1022
   
  --- Physical volume ---
  PV Name               /dev/sdc
  VG Name               vg01
  PV Size               1.00 GiB / not usable 0   
  Allocatable           yes             #提示已经在被使用中
  PE Size               1.00 MiB
  Total PE              1023
  Free PE               1022
  Allocated PE          1
  PV UUID               0MM2V8-QktI-c5rX-UwqA-Ebbo-nbT0-QpeSpC
   
  --- Physical Segments ---   #PE /dev/sdc 所在的LV
  Physical extent 0 to 0:
    Logical volume      /dev/vg01/lv01
    Logical extents     1023 to 1023
  Physical extent 1 to 1022:
    FREE
   
  --- Physical volume ---
  PV Name               /dev/sdd
  VG Name               vg01
  PV Size               1.00 GiB / not usable 0   
  Allocatable           yes 
  PE Size               1.00 MiB  #总容量
  Total PE              1023  #块数量
  Free PE               1023  #可用块
  Allocated PE          0
  PV UUID               JPBu9Y-U87N-xRwk-4qCA-rFMY-ciB9-plNnbr
   
  --- Physical Segments ---   #PE /dev/sdd 未被分配到LV 可用 pvmove 移动
  Physical extent 0 to 1022:
    FREE
[root@localhost /]# pvmove  /dev/sdd  #空PV不需要移动
  No data to move for vg01
[root@localhost /]# pvmove  /dev/sdb  #移动正在被使用中的LV的PV的数据
  /dev/sdb: Moved: 2.64%
  /dev/sdb: Moved: 100.00%
[root@localhost /]# pvdisplay -m      #移除 /dev/sdb 后的信息显示其已经可以从lv/vg中移除了
  ..................
  --- Physical volume ---
  PV Name               /dev/sdb
  VG Name               vg01
  PV Size               1.00 GiB / not usable 0   
  Allocatable           yes 
  PE Size               1.00 MiB
  Total PE              1023
  Free PE               1023
  Allocated PE          0
  PV UUID               bACZUQ-fSh3-UtGv-xcFQ-VLfr-GWcX-9wzBMd
   
  --- Physical Segments ---
  Physical extent 0 to 1022:
    FREE
  ..................
[root@localhost /]# vgreduce vg01  /dev/sdb 
  Removed "/dev/sdb" from volume group "vg01"
[root@localhost /]# pvscan 
  PV /dev/sda2   VG centos          lvm2 [23.51 GiB / 44.00 MiB free]
  PV /dev/sdc    VG vg01            lvm2 [1023.00 MiB / 1022.00 MiB free]
  PV /dev/sdd    VG vg01            lvm2 [1023.00 MiB / 0    free]
  PV /dev/sdb                       lvm2 [1.00 GiB]
  Total: 4 [26.51 GiB] / in use: 3 [25.51 GiB] / in no VG: 1 [1.00 GiB]
[root@localhost /]# pvremove  /dev/sdb
  Labels on physical volume "/dev/sdb" successfully wiped.
```
#### 删除 LV VG PV
```bash
[root@localhost /]# umount /dev/vg01/lv01
[root@localhost /]# vgchange -an /dev/vg01
  0 logical volume(s) in volume group "vg01" now active
[root@localhost /]# lvremove /dev/vg01/lv01
  Logical volume "lv01" successfully removed
[root@localhost /]# vgremove /dev/vg01
  Volume group "vg01" successfully removed
[root@localhost /]# pvremove /dev/sd{b,c,d}
  No PV label found on /dev/sdb.
  Labels on physical volume "/dev/sdc" successfully wiped.
  Labels on physical volume "/dev/sdd" successfully wiped.
```

#### LVM snapshot  
```bash
#镜像卷需与被镜像卷在相同卷组，快照将进行差异备份来提供还原,,因此需预先计划好镜像卷的空间用于存储差异数据
[root@localhost ~]# pvs
  PV         VG     Fmt  Attr PSize    PFree   
  /dev/sda2  centos lvm2 a--    23.51g   44.00m
  /dev/sdc   vg01   lvm2 a--  1023.00m 1022.00m
  /dev/sdd   vg01   lvm2 a--  1023.00m       0 
[root@localhost ~]# vgs
  VG     #PV #LV #SN Attr   VSize  VFree   
  centos   1   2   0 wz--n- 23.51g   44.00m
  vg01     2   1   0 wz--n-  2.00g 1022.00m
[root@localhost ~]# lvs
  LV   VG     Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root centos -wi-ao---- 21.46g                                                    
  swap centos -wi-ao----  2.00g                                                    
  lv01 vg01   -wi-ao----  1.00g 
[root@localhost ~]# lvcreate -L 100M -s -p r -n 'lv01-monitor' /dev/vg01/lv01   #-p r 权限只读（非必选） -s 创建快照
  Using default stripesize 64.00 KiB.
  Logical volume "lv01-monitor" created.
[root@localhost ~]# mkdir /lv01-tmp
[root@localhost ~]# mount /dev/vg01/lv01-monitor /lv01-tmp/
mount: /dev/mapper/vg01-lv01--monitor 写保护，将以只读方式挂载
[root@localhost /]# tar -czvf lv01_backup.tar.gz /lv01-tmp/* 
tar: 从成员名中删除开头的“/”
/lv01-tmp/lost+found/
[root@localhost /]# umount /lv01-tmp/
[root@localhost /]# lvremove /dev/vg01/lv01-monitor 
Do you really want to remove active logical volume vg01/lv01-monitor? [y/n]: y
  Logical volume "lv01-monitor" successfully removed
  
#还原快照数据：
[root@localhost /]# umount /dev/vg01/lv01 
[root@localhost /]# mkfs.ext4 /dev/vg01/lv01 
mke2fs 1.42.9 (28-Dec-2013)
文件系统标签=
OS type: Linux
块大小=4096 (log=2)
分块大小=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
65536 inodes, 262144 blocks
13107 blocks (5.00%) reserved for the super user
第一个数据块=0
Maximum filesystem blocks=268435456
8 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376

Allocating group tables: 完成                            
正在写入inode表: 完成                            
Creating journal (8192 blocks): 完成
Writing superblocks and filesystem accounting information: 完成

[root@localhost /]# mount /dev/vg01/lv01 /mnt
[root@localhost /]# tar -zxvf lv01
lv01_backup.tar.gz  lv01-tmp/           
[root@localhost /]# tar -zxvf lv01_backup.tar.gz -C /mnt/
lv01-tmp/lost+found/
```

















