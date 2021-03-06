#### RAID 0
```bash
[root@localhost ~]#  lsblk | grep "^sd*"                              
sda                          8:0    0   24G  0 disk 
sdb                          8:16   0    1G  0 disk 
sdc                          8:32   0    1G  0 disk 
sdd                          8:48   0    1G  0 disk 
sr0                         11:0    1 1024M  0 rom  
[root@localhost ~]# mdadm -C /dev/md0 -a yes -l 0 -n 3 /dev/sd{b,c,d} 
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
[root@localhost ~]# mdadm -D /dev/md0
/dev/md0:
        Version : 1.2
  Creation Time : Sun Nov 12 15:12:04 2017
     Raid Level : raid0
     Array Size : 3142656 (3.00 GiB 3.22 GB)
   Raid Devices : 3
  Total Devices : 3
    Persistence : Superblock is persistent

    Update Time : Sun Nov 12 15:12:04 2017
          State : clean 
 Active Devices : 3
Working Devices : 3
 Failed Devices : 0
  Spare Devices : 0

     Chunk Size : 512K

           Name : localhost.localdomain:0  (local to host localhost.localdomain)
           UUID : 39710d97:465e9ed5:d08ce8b3:376703bb
         Events : 0

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
```
#### RAID5
```bash
[root@localhost ~]# mdadm --stop /dev/md0  
mdadm: stopped /dev/md0
[root@localhost ~]# lsblk | grep "^sd*"
sda                          8:0    0   24G  0 disk 
sdb                          8:16   0    1G  0 disk 
sdc                          8:32   0    1G  0 disk 
sdd                          8:48   0    1G  0 disk 
sde                          8:64   0    1G  0 disk 
sdf                          8:80   0    1G  0 disk 
sdg                          8:96   0    1G  0 disk 
sr0                         11:0    1 1024M  0 rom  
[root@localhost ~]# mdadm -C /dev/md1 -a yes -l 5 -n 4 /dev/sd{b,c,d,e} -x 1 /dev/sdf
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md1 started.
[root@localhost ~]# cat /proc/mdstat 
Personalities : [raid6] [raid5] [raid4] 
md1 : active raid5 sde[5] sdf[4](S) sdd[2] sdc[1] sdb[0]
      3142656 blocks super 1.2 level 5, 512k chunk, algorithm 2 [4/4] [UUUU]
      
unused devices: <none>
[root@localhost ~]# mdadm -D /dev/md1
/dev/md1:
        Version : 1.2
  Creation Time : Sun Nov 12 15:23:02 2017
     Raid Level : raid5
     Array Size : 3142656 (3.00 GiB 3.22 GB)
  Used Dev Size : 1047552 (1023.00 MiB 1072.69 MB)
   Raid Devices : 4
  Total Devices : 5
    Persistence : Superblock is persistent

    Update Time : Sun Nov 12 15:23:09 2017
          State : clean 
 Active Devices : 4
Working Devices : 5
 Failed Devices : 0
  Spare Devices : 1

         Layout : left-symmetric
     Chunk Size : 512K

           Name : localhost.localdomain:1  (local to host localhost.localdomain)
           UUID : 464891ae:43d8467f:bac186bc:55002580
         Events : 18

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       5       8       64        3      active sync   /dev/sde

       4       8       80        -      spare   /dev/sdf
[root@localhost ~]# mdadm /dev/md1 --add /dev/sdg       #添加RAID5的热备盘/dev/sdg
mdadm: added /dev/sdg
[root@localhost ~]# mdadm -D /dev/md1            
/dev/md1:
        Version : 1.2
  Creation Time : Sun Nov 12 15:23:02 2017
     Raid Level : raid5
     Array Size : 3142656 (3.00 GiB 3.22 GB)
  Used Dev Size : 1047552 (1023.00 MiB 1072.69 MB)
   Raid Devices : 4
  Total Devices : 6
    Persistence : Superblock is persistent

    Update Time : Sun Nov 12 15:25:19 2017
          State : clean 
 Active Devices : 4
Working Devices : 6
 Failed Devices : 0
  Spare Devices : 2

         Layout : left-symmetric
     Chunk Size : 512K

           Name : localhost.localdomain:1  (local to host localhost.localdomain)
           UUID : 464891ae:43d8467f:bac186bc:55002580
         Events : 19

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       5       8       64        3      active sync   /dev/sde

       4       8       80        -      spare   /dev/sdf
       6       8       96        -      spare   /dev/sdg      #  <--------------- 
[root@localhost ~]# mdadm -G /dev/md1 -n 5                    # 指定RAID5的工作设备数量（将从热备硬盘中添加）
mdadm: Need to backup 6144K of critical section..
[root@localhost ~]# mdadm -D /dev/md1     
/dev/md1:
        Version : 1.2
  Creation Time : Sun Nov 12 15:23:02 2017
     Raid Level : raid5
     Array Size : 3142656 (3.00 GiB 3.22 GB)
  Used Dev Size : 1047552 (1023.00 MiB 1072.69 MB)
   Raid Devices : 5
  Total Devices : 6
    Persistence : Superblock is persistent

    Update Time : Sun Nov 12 15:26:53 2017
          State : clean, reshaping 
 Active Devices : 5
Working Devices : 6
 Failed Devices : 0
  Spare Devices : 1

         Layout : left-symmetric
     Chunk Size : 512K

 Reshape Status : 27% complete
  Delta Devices : 1, (4->5)

           Name : localhost.localdomain:1  (local to host localhost.localdomain)
           UUID : 464891ae:43d8467f:bac186bc:55002580
         Events : 45

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       5       8       64        3      active sync   /dev/sde
       6       8       96        4      active sync   /dev/sdg

       4       8       80        -      spare   /dev/sdf
[root@localhost ~]# mdadm /dev/md1 --fail /dev/sdb              #模拟RAID5下的设备损坏并将其删除
mdadm: set /dev/sdb faulty in /dev/md1
[root@localhost ~]# mdadm /dev/md1 --remove /dev/sdb 
mdadm: hot removed /dev/sdb from /dev/md1
[root@localhost ~]# mdadm -D /dev/md1
/dev/md1:
        Version : 1.2
  Creation Time : Sun Nov 12 15:23:02 2017
     Raid Level : raid5
     Array Size : 4190208 (4.00 GiB 4.29 GB)
  Used Dev Size : 1047552 (1023.00 MiB 1072.69 MB)
   Raid Devices : 5
  Total Devices : 4
    Persistence : Superblock is persistent

    Update Time : Sun Nov 12 15:32:54 2017
          State : clean, degraded 
 Active Devices : 4
Working Devices : 4
 Failed Devices : 0
  Spare Devices : 0

         Layout : left-symmetric
     Chunk Size : 512K

           Name : localhost.localdomain:1  (local to host localhost.localdomain)
           UUID : 464891ae:43d8467f:bac186bc:55002580
         Events : 72

    Number   Major   Minor   RaidDevice State
       4       8       80        0      active sync   /dev/sdf
       -       0        0        1      removed
       2       8       48        2      active sync   /dev/sdd
       5       8       64        3      active sync   /dev/sde
       6       8       96        4      active sync   /dev/sdg
[root@localhost ~]# cat /proc/mdstat 
Personalities : [raid6] [raid5] [raid4] 
md1 : active raid5 sdg[6] sde[5] sdf[4] sdd[2]
      4190208 blocks super 1.2 level 5, 512k chunk, algorithm 2 [5/4] [U_UUU]
```
#### 模拟故障并将故障设备从阵列删除后销毁阵列
```bash
[root@localhost ~]# mdadm  /dev/md1 --fail /dev/sdd --remove /dev/sdd
mdadm: set /dev/sdd faulty in /dev/md1
mdadm: hot removed /dev/sdd from /dev/md1
[root@localhost ~]# mdadm  /dev/md1 --fail /dev/sde --remove /dev/sde
mdadm: set /dev/sde faulty in /dev/md1
mdadm: hot removed /dev/sde from /dev/md1
[root@localhost ~]# mdadm  /dev/md1 --fail /dev/sdf --remove /dev/sdf
mdadm: set /dev/sdf faulty in /dev/md1
mdadm: hot removed /dev/sdf from /dev/md1
[root@localhost ~]# mdadm  /dev/md1 --fail /dev/sdg --remove /dev/sdg 
mdadm: set /dev/sdg faulty in /dev/md1
mdadm: hot removed /dev/sdg from /dev/md1
[root@localhost ~]# mdadm --stop /dev/md1               #停止指定阵列并释放磁盘
mdadm: stopped /dev/md1
[root@localhost ~]# cat /proc/mdstat       
Personalities : [raid6] [raid5] [raid4] 
unused devices: <none>
```
#### 创建后拆除阵列以模拟硬盘更换主机后的阵列重装配
```bash
[root@localhost ~]# mdadm -C /dev/md5 -a yes -l 5 -n 4 /dev/sd{b,c,d,e} -x 1 /dev/sdf
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md5 started.
[root@localhost ~]# cat /proc/mdstat 
Personalities : [raid6] [raid5] [raid4] 
md5 : active raid5 sde[5] sdf[4](S) sdd[2] sdc[1] sdb[0]
      3142656 blocks super 1.2 level 5, 512k chunk, algorithm 2 [4/4] [UUUU]
      
unused devices: <none>
[root@localhost ~]# mdadm --stop /dev/md5                               #停止阵列
mdadm: stopped /dev/md5
[root@localhost ~]# mdadm -A /dev/md5 /dev/sd{b,c,d,e,f}                #重新装配
mdadm: /dev/md5 has been started with 4 drives and 1 spare.
[root@localhost ~]# cat /proc/mdstat 
Personalities : [raid6] [raid5] [raid4] 
md5 : active raid5 sdb[0] sdf[4](S) sde[5] sdd[2] sdc[1]
      3142656 blocks super 1.2 level 5, 512k chunk, algorithm 2 [4/4] [UUUU]    #4个U，都在线
      
unused devices: <none>
```
#### 创建配置文件
```bash
[root@localhost ~]# mdadm -Ds > /etc/mdadm.conf
[root@localhost ~]# cat /etc/mdadm.conf 
ARRAY /dev/md5 metadata=1.2 spares=1 name=localhost.localdomain:5 UUID=95ea9008:1f7bae13:b3bf6f23:889b03ae
```
#### RAID10
```bash
mdadm -C /dev/md0 -l1 -n2 /dev/sdb /dev/sdc
mdadm -C /dev/md1 -l1 -n2 /dev/sdd /dev/sde
mdadm -C /dev/md2 -l1 -n2 /dev/sdf /dev/sdg
mdadm -C /dev/md3 -l0 -n3 /dev/md0 /dev/md1 /dev/md2
```
