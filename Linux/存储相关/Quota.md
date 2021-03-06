#### 例
```bash
[root@localhost /]# mke2fs -t ext4 -L quota-test-volume /dev/sdb        #格式化磁盘，创建quota环境
mke2fs 1.42.9 (28-Dec-2013)
/dev/sdb is entire device, not just one partition!
无论如何也要继续? (y,n) y
文件系统标签=quota-test-volum
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
[root@localhost /]# mount -t ext4 -o usrquota,grpquota /dev/sdb /mnt    #挂载并加入Quota对用户及组限制的参数
[root@localhost /]# cat >> /etc/fstab <<eof                             #对需限制的设备加入{usr,grp}quota参数
> /dev/sdb    /mnt    ext4    default,usrquota,grpquota 0 0
> eof
[root@localhost /]# quotacheck -avug  #扫描/etc/fstab中加入{usr,grp}quota的设备并在其根目录产生quota{user,group}
quotacheck: Your kernel probably supports journaled quota but you are not using it. Consider switching to \
journaled quota to avoid running quotacheck after an unclean shutdown.
quotacheck: Scanning /dev/sdb [/mnt] done
quotacheck: Cannot stat old user quota file /mnt/aquota.user: 没有那个文件或目录. Usage will not be subtracted.
quotacheck: Cannot stat old group quota file /mnt/aquota.group: 没有那个文件或目录. Usage will not be subtracted.
quotacheck: Cannot stat old user quota file /mnt/aquota.user: 没有那个文件或目录. Usage will not be subtracted.
quotacheck: Cannot stat old group quota file /mnt/aquota.group: 没有那个文件或目录. Usage will not be subtracted.
quotacheck: Checked 2 directories and 0 files
quotacheck: Old file not found.
quotacheck: Old file not found.
[root@localhost /]# ll /mnt/
总用量 32
-rw------- 1 root root  6144 11月 10 20:04 aquota.group
-rw------- 1 root root  6144 11月 10 20:04 aquota.user
drwx------ 2 root root 16384 11月 10 19:59 lost+found
[root@localhost /]# quotaon -avug    #启动所有支持quota.{user,group}限制的FS（针对指定的U/G限制：quotaon -vug /Path）
/dev/sdb [/mnt]: group quotas turned on
/dev/sdb [/mnt]: user quotas turned on
```

#### 限额设置
```bash
[root@localhost /]# edquota -t                  #设置宽限时间
Grace period before enforcing soft limits for users:
Time units may be: days, hours, minutes, or seconds
  Filesystem             Block grace period     Inode grace period
  /dev/sdb                      7days                  7days
[root@localhost /]# edquota -u root             #针对root用户设置限额（针对组：edquota -g <groupname>）
Disk quotas for user root (uid 0):
  Filesystem                   blocks       soft       hard     inodes     soft     hard
  /dev/sdb                         20       1024000    2048000       2        0        0
[root@localhost /]# edquota -p root -u nginx    #将对root用户的限额设置拷贝给nginx用户
[root@localhost /]# quotaon -avug               #启用所有的Quota限制（指定限制：quota -vug </Path>）
quotaon: using /mnt/aquota.group on /dev/sdb [/mnt]: 设备或资源忙
quotaon: using /mnt/aquota.user on /dev/sdb [/mnt]: 设备或资源忙
[root@localhost /]# quotaoff -a                 #若报错先关闭Quota在打开
[root@localhost /]# quotaon -avug
/dev/sdb [/mnt]: group quotas turned on
/dev/sdb [/mnt]: user quotas turned on
[root@localhost /]# repquota -avugs             #查看所有（详细信息）用户和组的限额信息与使用状态
*** Report for user quotas on device /dev/sdb
Block grace time: 7days; Inode grace time: 7days
                        Space limits                File limits
User            used    soft    hard  grace    used  soft  hard  grace
----------------------------------------------------------------------
root      --     20K   1000M   2000M              2     0     0       
nginx     --      0K   1000M   2000M              0     0     0       

Statistics:
Total blocks: 7
Data blocks: 1
Entries: 2
Used average: 2.000000

*** Report for group quotas on device /dev/sdb
Block grace time: 7days; Inode grace time: 7days
                        Space limits                File limits
Group           used    soft    hard  grace    used  soft  hard  grace
----------------------------------------------------------------------
root      --     20K      0K      0K              2     0     0       

Statistics:
Total blocks: 6
Data blocks: 1
Entries: 1
Used average: 1.000000
```
#### 测试
```bash
[root@localhost mnt]# dd if=/dev/zero of=./test.file count=1024 bs=1M
dd: 写入"./test.file" 出错: 设备上没有空间
记录了958+0 的读入
记录了957+0 的写出
1003880448字节(1.0 GB)已复制，4.57325 秒，220 MB/秒
[root@localhost mnt]# repquota -vus /mnt 
*** Report for user quotas on device /dev/sdb
Block grace time: 7days; Inode grace time: 7days
                        Space limits                File limits
User            used    soft    hard  grace    used  soft  hard  grace
----------------------------------------------------------------------
root      --    958M   1000M   2000M              3     0     0       
nginx     --      0K   1000M   2000M              0     0     0       

Statistics:
Total blocks: 7
Data blocks: 1
Entries: 2
Used average: 2.000000
```
#### Other
```txt
-a 所有        quotaon -avug
-u 用户限制    quotaon -au /data
-g 组限制      quotaoff -ug /date
```
