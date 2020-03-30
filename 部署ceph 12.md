## 一、概念介绍

- Ceph 简介

Ceph 是一个开源的分布式存储系统，设计初衷是提供较好的性能、可靠性和可扩展性。 Ceph 独一无二地在一个统一的系统中同时提供了对象、块、和文件存储功能。 Ceph 消除了对系统单一中心节点的依赖，实现了无中心结构的设计思想。



- Ceph 包括多个组件：


Ceph Monitors(MON)：负责生成集群票选机制。所有的集群节点都会向 Mon 进行汇报，并在每次状态变更时进行共享信息。

Ceph Object Store Devices(OSD)：负责在本地文件系统保存对象，并通过网络提供访问。通常 OSD 守护进程会绑定在集群的一个物理盘上，Ceph 客户端直接和 OSD 打交道。

Ceph Manager(MGR)：提供额外的监控和界面给外部的监管系统使用。

Reliable Autonomic Distributed Object Stores：Ceph 存储集群的核心。这一层用于为存储数据提供一致性保障，执行数据复制、故障检测以及恢复等任务。



MDS：全称 Ceph Metadata Server，元数据服务器，只有 Ceph FS 需要它。



MON：MON通过保存一系列集群状态 map 来监视集群的组件，使用 map 保存集群的状态，为了防止单点故障，因此 monitor 的服务器需要奇数台（大于等于 3 台），如果出现意见分歧，采用投票机制，少数服从多数。



Object：Ceph 最底层的存储单元是 Object 对象，每个 Object 包含元数据和原始数据。



CRUSH：是Ceph使用的数据分布算法，类似一致性哈希，让数据分配到预期的地方。



RBD：全称 RADOS block device，它是 RADOS 块设备，对外提供块存储服务。



RGW：全称 RADOS gateway，RADOS网关，提供对象存储，接口与 S3 和 Swift 兼容。



CephFS：提供文件系统级别的存储。



为了在 Ceph 上进行读写，客户端首先要联系 MON，获取最新的集群地图，其中包含了集群拓扑以及数据存储位置的信息。Ceph 客户端使用集群地图来获知需要交互的 OSD，从而和特定 OSD 建立联系





## 二、平台介绍

- 操作系统：CentOS 7.7

- 节点列表

  | 主机名  | IP              | 网卡   | 磁盘                 | 功能说明              |
  | ------- | --------------- | ------ | -------------------- | --------------------- |
  | node1   | 192.168.255.196 | 单网卡 | 除系统盘，新加2个20G | mon，osd，mgr，deploy |
  | node2   | 192.168.255.197 | 单网卡 | 除系统盘，新加2个20G | mon，osd，mgr         |
  | node3   | 192.168.255.198 | 单网卡 | 除系统盘，新加2个20G | mon，osd，mgr         |
  | client  | 192.168.255.199 | 单网卡 | 系统盘               | 文件系统client        |
  | client2 | 192.168.255.200 | 单网卡 | 系统盘               | RBD client            |

  

  

- 关闭防火墙和selinux

  ```bash
  # 查看防火墙状态
  # firewall-cmd --state
  
  # 关闭防火墙
  # systemctl stop firewalld.service
  # systemctl disable firewalld.service 
  
  # 关闭seliinux
  # sed -i 's/enforcing/disabled/g' /etc/sysconfig/selinux
  
  ```



## 三、前期工作

说明：所有节点执行

3.1、配置hosts文件

```bash
# vi /etc/hosts
192.168.255.196 node1
192.168.255.197 node2
192.168.255.198 node3
```



3.2、配置yum

```bash
# cat /etc/yum.repos.d/ceph.repo 
[ceph]

name=ceph

baseurl=http://mirrors.163.com/ceph/rpm-luminous/el7/x86_64/

gpgckeck=0

gpgkey=http://mirrors.163.com/ceph/keys/release.asc

[ceph-noarch]

name=Ceph noarch packages

baseurl=http://mirrors.163.com/ceph/rpm-luminous/el7/noarch/

gpgcheck=0

gpgkey=http://mirrors.163.com/ceph/keys/release.asc

# yum makecache

# 安装最新epel源
# yum -y install epel-release
```



3.3、配置python环境

```bash
配置python2.7环境

#wget https://files.pythonhosted.org/packages/ed/69/c805067de1feedbb98c53174b0f2df44cc05e0e9ee73bb85eebc59e508c6/setuptools-41.0.0.zip
#wget https://files.pythonhosted.org/packages/36/fa/51ca4d57392e2f69397cd6e5af23da2a8d37884a605f9e3f2d3bfdc48397/pip-19.0.3.tar.gz
#unzip setuptools-41.0.0.zip 
#cd setuptools-41.0.0
#python setup.py install
#tar zxf pip-19.0.3.tar.gz 
#cd pip-19.0.3
#python setup.py install
```







## 四、安装配置ceph

4.1、node1安装deploy ceph

```bash
[root@node1 ~]# yum -y install ceph-deploy
[root@node1 ~]# yum -y install ceph
```



4.2、node2、node3安装ceph

```bash
[root@node2 ~]# yum -y install ceph

[root@node3 ~]# yum -y install ceph
```



4.3、所有节点执行ceph -v 返回版本信息

```bash
# ceph -v
ceph version 12.2.13 (584a20eb0237c657dc0567da126be145106aa47e) luminous (stable)
```



4.4、在 deploy节点配置节点间SSH免秘钥互联

```bash
[root@node1 ~]# ssh-keygen  -t rsa
[root@node1 ~]# ssh-copy-id -i node1   #输入节点密码 


[root@node1 ~]# ssh-copy-id -i node2   #输入节点密码 

 
[root@node1 ~]# ssh-copy-id -i node3   #输入节点密码 

[root@node1 ~]# ssh node1 date
Mon Mar 30 11:48:29 CST 2020
[root@node1 ~]# ssh node2 date
Mon Mar 30 11:48:32 CST 2020
[root@node1 ~]# ssh node3 date
Mon Mar 30 11:48:34 CST 2020
```



4.5、所有节点创建/etc/ceph目录（若不存在）

```bash
# mkdir -p  /ceph-install 
```



4.6、在deploy节点创建集群信息

```bash
[root@node1 ~]# cd  /ceph-install 

[root@node1 ceph]# ceph-deploy new node1 node2 node3    # 创建ceph集群
... ... 
[node3][DEBUG ] connected to host: node3 
[node3][DEBUG ] detect platform information from remote host
[node3][DEBUG ] detect machine type
[node3][DEBUG ] find the location of an executable
[node3][INFO  ] Running command: /usr/sbin/ip link show
[node3][INFO  ] Running command: /usr/sbin/ip addr show
[node3][DEBUG ] IP addresses found: [u'192.168.255.198']
[ceph_deploy.new][DEBUG ] Resolving host node3
[ceph_deploy.new][DEBUG ] Monitor node3 at 192.168.255.198
[ceph_deploy.new][DEBUG ] Monitor initial members are ['node1', 'node2', 'node3']
[ceph_deploy.new][DEBUG ] Monitor addrs are ['192.168.255.196', '192.168.255.197', '192.168.255.198']
[ceph_deploy.new][DEBUG ] Creating a random mon key...
[ceph_deploy.new][DEBUG ] Writing monitor keyring to ceph.mon.keyring...
[ceph_deploy.new][DEBUG ] Writing initial config to ceph.conf...

# 执行成功后查看
[root@node1 ceph]# ll
total 20
-rw-r--r-- 1 root root  244 Mar 27 17:51 ceph.conf
-rw-r--r-- 1 root root 4879 Mar 27 17:51 ceph-deploy-ceph.log
-rw------- 1 root root   73 Mar 27 17:51 ceph.mon.keyring
-rw-r--r-- 1 root root   92 Jan 31 05:37 rbdmap
```



4.7、部署mon

```bash
[root@node1 ceph]# ceph-deploy --overwrite-conf mon create-initial   #监控在/ceph-install 目录下执行，秘钥文件也会保存在这个目录
... ...
[node1][INFO  ] Running command: /usr/bin/ceph --connect-timeout=25 --cluster=ceph --name mon. --keyring=/var/lib/ceph/mon/ceph-node1/keyring auth get client.bootstrap-rgw
[node1][INFO  ] Running command: /usr/bin/ceph --connect-timeout=25 --cluster=ceph --name mon. --keyring=/var/lib/ceph/mon/ceph-node1/keyring auth get-or-create client.bootstrap-rgw mon allow profile bootstrap-rgw
[ceph_deploy.gatherkeys][INFO  ] Storing ceph.client.admin.keyring
[ceph_deploy.gatherkeys][INFO  ] Storing ceph.bootstrap-mds.keyring
[ceph_deploy.gatherkeys][INFO  ] Storing ceph.bootstrap-mgr.keyring
[ceph_deploy.gatherkeys][INFO  ] keyring 'ceph.mon.keyring' already exists
[ceph_deploy.gatherkeys][INFO  ] Storing ceph.bootstrap-osd.keyring
[ceph_deploy.gatherkeys][INFO  ] Storing ceph.bootstrap-rgw.keyring
[ceph_deploy.gatherkeys][INFO  ] Destroy temp directory /tmp/tmpbu4hwi

#将密钥文件复制到/etc/ceph目录下，才能正常执行ceph -s命令
[root@node1 ceph-install]# cp *.keyring /etc/ceph
```





4.8、查看集群状态

```bash
[root@node1 ceph-install]# ceph -s
  cluster:
    id:     4d819a21-f2ce-482f-9fbe-85c0c58916ab
    health: HEALTH_WARN
            clock skew detected on mon.node2, mon.node3
 
  services:
    mon: 3 daemons, quorum node1,node2,node3
    mgr: no daemons active
    osd: 0 osds: 0 up, 0 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0B
    usage:   0B used, 0B / 0B avail
    pgs:     
 
  
  #可以看到services中mon开启3个分别是node1 node2 node3
```



4.9、部署osd

部署节点上的osd 

```bash
[root@node1 ceph-install]# ceph-deploy osd create node1 --data /dev/sdb 
[root@node1 ceph-install]# ceph-deploy osd create node1 --data /dev/sdc
[root@node1 ceph-install]# ceph-deploy osd create node2 --data /dev/sdb
[root@node1 ceph-install]# ceph-deploy osd create node2 --data /dev/sdc
[root@node1 ceph-install]# ceph-deploy osd create node3 --data /dev/sdb
[root@node1 ceph-install]# ceph-deploy osd create node3 --data /dev/sdc
```





4.10、把配置和admin密钥放到各个节点

```bash
[root@node1 ceph-install]# ceph-deploy --overwrite-conf admin node1 node2 node3
```



4.11、部署mgr

```bash
[root@node1 ceph-install]# ceph-deploy mgr create node1 node2 node3
```







查看ceph状态

```bash
[root@node1 ceph-install]# ceph -s   #在services 看到osd 共6个，且均状态up in
  cluster:
    id:     4d819a21-f2ce-482f-9fbe-85c0c58916ab
    health: HEALTH_WARN
            clock skew detected on mon.node2, mon.node3
 
  services:
    mon: 3 daemons, quorum node1,node2,node3
    mgr: node1(active), standbys: node2, node3
    osd: 6 osds: 6 up, 6 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0B
    usage:   6.02GiB used, 114GiB / 120GiB avail
    pgs:     
   
```



查看osd状态 

```bash
[root@node1 ceph-install]# ceph osd tree
ID CLASS WEIGHT  TYPE NAME      STATUS REWEIGHT PRI-AFF 
-1       0.11691 root default                           
-3       0.03897     host node1                         
 0   hdd 0.01949         osd.0      up  1.00000 1.00000 
 1   hdd 0.01949         osd.1      up  1.00000 1.00000 
-5       0.03897     host node2                         
 2   hdd 0.01949         osd.2      up  1.00000 1.00000 
 3   hdd 0.01949         osd.3      up  1.00000 1.00000 
-7       0.03897     host node3                         
 4   hdd 0.01949         osd.4      up  1.00000 1.00000 
 5   hdd 0.01949         osd.5      up  1.00000 1.00000 
```





4.10、部署mds信息

```bash
[root@node1 ceph]# ceph-deploy --overwrite-conf mds create node1 node2 node3
... ...
[ceph_deploy.mds][DEBUG ] deploying mds bootstrap to node3
[node3][DEBUG ] write cluster configuration to /etc/ceph/{cluster}.conf
[node3][WARNIN] mds keyring does not exist yet, creating one
[node3][DEBUG ] create a keyring file
[node3][DEBUG ] create path if it doesn't exist
[node3][INFO  ] Running command: ceph --cluster ceph --name client.bootstrap-mds --keyring /var/lib/ceph/bootstrap-mds/ceph.keyring auth get-or-create mds.node3 osd allow rwx mds allow mon allow profile mds -o /var/lib/ceph/mds/ceph-node3/keyring
[node3][INFO  ] Running command: systemctl enable ceph-mds@node3
[node3][WARNIN] Created symlink from /etc/systemd/system/ceph-mds.target.wants/ceph-mds@node3.service to /usr/lib/systemd/system/ceph-mds@.service.
[node3][INFO  ] Running command: systemctl start ceph-mds@node3
[node3][INFO  ] Running command: systemctl enable ceph.target
```



4.11、查看ceph信息

```bash
[root@node1 ceph-install]# ceph -s  #能看到总大小
  cluster:
    id:     4d819a21-f2ce-482f-9fbe-85c0c58916ab
    health: HEALTH_WARN
            clock skew detected on mon.node2, mon.node3
 
  services:
    mon: 3 daemons, quorum node1,node2,node3
    mgr: node1(active), standbys: node2, node3
    osd: 6 osds: 6 up, 6 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0B
    usage:   6.02GiB used, 114GiB / 120GiB avail
    pgs: 
```





4.12、创建文件系统ceph

```bash
# 创建poll存储池和名为data的文件系统
[root@node1 ceph-install]# ceph osd pool create data_data 32
pool 'data_data' created

[root@node1 ceph-install]# ceph osd pool create data_metadata 32
pool 'data_metadata' created

[root@node1 ceph-install]# ceph fs new data data_metadata data_data
new fs with metadata pool 2 and data pool 1

# 查看存储池和文件系统
[root@node1 ceph-install]# ceph osd pool ls
data_data
data_metadata
[root@node1 ceph-install]# ceph fs ls
name: data, metadata pool: data_metadata, data pools: [data_data ]
```





## 五、客户端文件挂载配置

5.1、客户端安装ceph-fuser

```bash
[root@client ~]# yum -y install ceph-fuse
```



5.2、客户端挂载

```bash
[root@client ~]# mkdir /client
[root@client ~]# mkdir -p /etc/ceph

[root@client ~]# scp 192.168.255.196:/etc/ceph/ceph.conf /etc/ceph/
[root@client ~]# scp 192.168.255.196:/etc/ceph/ceph.client.admin.keyring /etc/ceph/

#执行挂载
[root@client ~]# ceph-fuse -m 192.168.255.196,192.168.255.197,192.168.255.198:6789 /client

效果
[root@client ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
... ...
ceph-fuse                 36G     0   36G   0% /client
```





## 六、RDB块存储方式

6.1、在node1上创建存储池

```bash
[root@node1 ceph]# ceph osd pool create cephrbd 256
pool 'cephrbd' created
```



6.2、创建名伟image的镜像大小10G

```bash
[root@node1 ceph]# rbd create cephrbd/image --image-feature layering --size 10G
```



6.3、查看池中镜像及信息

```bash
[root@node1 ceph]# rbd ls cephrbd
image
[root@node1 ceph]# rbd info cephrbd/image
rbd image 'image':
        size 10GiB in 2560 objects
        order 22 (4MiB objects)
        block_name_prefix: rbd_data.105d6b8b4567
        format: 2
        features: layering
        flags: 
        create_timestamp: Mon Mar 30 16:00:55 2020
```



6.4、如果镜像扩容

```bash
[root@node1 ceph]#  rbd resize --size 20G cephrbd/image  
Resizing image: 100% complete...done.
```



6.5、镜像缩容

```bash
#  rbd resize --size 12G cephrbd/image --allow-shrink
Resizing image: 100% complete...done.
```



6.6、删除镜像

```bash
# rbd rm cephrbd/demo-img
```



6.7、rbd客户端挂载使用磁盘镜像

- ```bash
  # 安装依赖
  # yum install -y yum-utils && \
  yum-config-manager --add-repo https://dl.fedoraproject.org/pub/epel/7/x86_64/ && \
  yum install --nogpgcheck -y epel-release && \
  rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7 && \
  rm -f /etc/yum.repos.d/dl.fedoraproject.org*
  
  # 客户端安装ceph-common包
  [root@client2 ~]#  yum -y install ceph-common
  
  #拷贝节点1的ceph.conf等文件到客户端
  [root@client2 etc]# scp 192.168.255.196:/etc/ceph/ceph.conf /etc/ceph/
  [root@client2 etc]# scp 192.168.255.196:/etc/ceph/ceph.client.admin.keyring /etc/ceph/
  
  
  #执行ceph -s客户机上查看集群
  [root@client2 etc]# ceph -s
    cluster:
      id:     4d819a21-f2ce-482f-9fbe-85c0c58916ab
      health: HEALTH_WARN
              application not enabled on 1 pool(s)
              clock skew detected on mon.node2, mon.node3
   
    services:
      mon: 3 daemons, quorum node1,node2,node3
      mgr: node1(active), standbys: node2, node3
      mds: data-1/1/1 up  {0=node1=up:active}, 2 up:standby
      osd: 6 osds: 6 up, 6 in
   
    data:
      pools:   3 pools, 320 pgs
      objects: 27 objects, 42.8KiB
      usage:   6.03GiB used, 114GiB / 120GiB avail
      pgs:     320 active+clean
   
    io:
      client:   0B/s rd, 0op/s rd, 0op/s wr
      
      
      
  # 执行命令防止镜像missing required protocol freatures失败
   
  [root@client2 etc]# ceph osd crush tunables hammer
  adjusted tunables profile to hammer
  
  
  # 执行镜像挂载
  [root@client2 etc]# rbd map cephrbd/image
  /dev/rbd0
  [root@client2 etc]# lsblk
  NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
  sda               8:0    0   40G  0 disk 
  ├─sda1            8:1    0    1G  0 part /boot
  └─sda2            8:2    0   39G  0 part 
    ├─centos-root 253:0    0 35.2G  0 lvm  /
    └─centos-swap 253:1    0  3.8G  0 lvm  [SWAP]
  sr0              11:0    1 1024M  0 rom  
  rbd0            252:0    0   12G  0 disk
  
  
  # 查看磁盘映射信息
  [root@client2 etc]# rbd showmapped
  id pool    image snap device    
  0  cephrbd image -    /dev/rbd0 
  
  
  #格式化、挂载/dev/rbd0
  [root@client2 etc]# mkfs.ext4 /dev/rbd0
  
  [root@client2 etc]# mkdir /image
  [root@client2 etc]# mount /dev/rbd0 /image
  [root@client2 etc]# df -h
  Filesystem               Size  Used Avail Use% Mounted on
  ... ...
  /dev/rbd0                 12G   41M   12G   1% /image
  
  #设置开机自启动
  [root@client2 etc]# echo -e "/dev/rbd0 /image ext4 defaults 0 0" >> /etc/fstab
  
  #为镜像创建快照
  [root@client2 etc]# rbd snap create cephrbd/image --snap image-sn1
  
  #查看快照
  [root@client2 etc]# rbd snap ls cephrbd/image
  SNAPID NAME       SIZE TIMESTAMP                
       4 image-sn1 12GiB Mon Mar 30 16:54:54 2020 
       
       
  #删除快照
  [root@client2 etc]# rbd snap remove cephrbd/image@image-sn1
  Removing snap: 100% complete...done.
  
  
  ```



6.8、rbd在线扩容

以下均是在 rbd客户端操作，如果向从12G扩容到20G

```bash
[root@client2 etc]# rbd showmapped     # 查看设备
id pool    image snap device    
0  cephrbd image -    /dev/rbd0 
[root@client2 etc]# rbd info cephrbd/image
rbd image 'image':
        size 12GiB in 3072 objects
        order 22 (4MiB objects)
        block_name_prefix: rbd_data.105d6b8b4567
        format: 2
        features: layering
        flags: 
        create_timestamp: Mon Mar 30 16:00:55 2020
[root@client2 etc]# rbd resize --size 20G cephrbd/image   #扩容
Resizing image: 100% complete...done.
[root@client2 etc]# resize2fs /dev/rbd0                   #扩容格式化
resize2fs 1.42.9 (28-Dec-2013)
Filesystem at /dev/rbd0 is mounted on /image; on-line resizing required
old_desc_blocks = 2, new_desc_blocks = 3
The filesystem on /dev/rbd0 is now 5242880 blocks long.

[root@client2 etc]# df -h
Filesystem               Size  Used Avail Use% Mounted on
... ...
/dev/rbd0                 20G   44M   19G   1% /image
```





## 七、dashboard配置

7.1、在node1上执行以下命令

```bash
[root@node1 ceph]#  ceph mgr module enable dashboard
[root@node1 ceph]# ceph config-key set mgr/dashboard/node1/server_addr 192.168.255.196
set mgr/dashboard/node1/server_addr
```



7.2、查看效果

浏览器输入

http://192.168.255.196:7000/



![企业微信截图_20200330152523.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gdbzh890esj31h508ptfi.jpg)![企业微信截图_20200330152243.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gdbzedis3qj31he0l4at7.jpg)

![企业微信截图_20200330152534.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gdbzhn04ghj31hb0a946z.jpg)