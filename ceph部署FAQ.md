Q1:执行加入osd磁盘的时候，报以下提示：

```bash
#  ceph-deploy osd create --data /dev/vdb ceph-1 
[ceph_deploy.conf][DEBUG ] found configuration file at: /root/.cephdeploy.conf
[ceph_deploy.cli][INFO  ] Invoked (2.0.1): /usr/bin/ceph-deploy osd create --data /dev/vdb ceph-1
[ceph_deploy.cli][INFO  ] ceph-deploy options:
[ceph_deploy.cli][INFO  ]  verbose                       : False
[ceph_deploy.cli][INFO  ]  bluestore                     : None
[ceph_deploy.cli][INFO  ]  cd_conf                       : <ceph_deploy.conf.cephdeploy.Conf instance at 0x7ff724930e60>
[ceph_deploy.cli][INFO  ]  cluster                       : ceph
[ceph_deploy.cli][INFO  ]  fs_type                       : xfs
[ceph_deploy.cli][INFO  ]  block_wal                     : None
[ceph_deploy.cli][INFO  ]  default_release               : False
[ceph_deploy.cli][INFO  ]  username                      : None
[ceph_deploy.cli][INFO  ]  journal                       : None
[ceph_deploy.cli][INFO  ]  subcommand                    : create
[ceph_deploy.cli][INFO  ]  host                          : ceph-1
[ceph_deploy.cli][INFO  ]  filestore                     : None
[ceph_deploy.cli][INFO  ]  func                          : <function osd at 0x7ff724907398>
[ceph_deploy.cli][INFO  ]  ceph_conf                     : None
[ceph_deploy.cli][INFO  ]  zap_disk                      : False
[ceph_deploy.cli][INFO  ]  data                          : /dev/vdb
[ceph_deploy.cli][INFO  ]  block_db                      : None
[ceph_deploy.cli][INFO  ]  dmcrypt                       : False
[ceph_deploy.cli][INFO  ]  overwrite_conf                : False
[ceph_deploy.cli][INFO  ]  dmcrypt_key_dir               : /etc/ceph/dmcrypt-keys
[ceph_deploy.cli][INFO  ]  quiet                         : False
[ceph_deploy.cli][INFO  ]  debug                         : False
[ceph_deploy.osd][DEBUG ] Creating OSD on cluster ceph with data device /dev/vdb
[ceph_deploy][ERROR ] RuntimeError: bootstrap-osd keyring not found; run 'gatherkeys'
```



A1:解决办法

```bash
# ceph-deploy gatherkeys  ceph-1  
```













Q2:无法添加osd磁盘 



A2：解除ceph对磁盘的占用

```bash
# lsblk 
NAME                                                                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0                                                                      11:0    1 1024M  0 rom  
vda                                                                     253:0    0  100G  0 disk 
├─vda1                                                                  253:1    0    1G  0 part /boot
├─vda2                                                                  253:2    0  7.9G  0 part [SWAP]
└─vda3                                                                  253:3    0 91.1G  0 part /
vdb                                                                     253:16   0    1T  0 disk 
└─ceph--587b3b32--c81a--49ed--8a94--cb7ca9617efb-osd--block--d11ea64c--ac20--4889--be64--c245859e2063
                                                                        252:1    0 1024G  0 lvm  
vdc                                                                     253:32   0    1T  0 disk 
└─ceph--6c1ebfcb--45a2--4f11--832b--bd9bb15ebedc-osd--block--ae3b152e--dc18--412f--b5fc--775ca8bdb738
                                                                        252:0    0 1024G  0 lvm  
[root@ceph-1 ~]# lsblk
NAME                                                                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0                                                                      11:0    1 1024M  0 rom  
vda                                                                     253:0    0  100G  0 disk 
├─vda1                                                                  253:1    0    1G  0 part /boot
├─vda2                                                                  253:2    0  7.9G  0 part [SWAP]
└─vda3                                                                  253:3    0 91.1G  0 part /
vdb                                                                     253:16   0    1T  0 disk 
└─ceph--587b3b32--c81a--49ed--8a94--cb7ca9617efb-osd--block--d11ea64c--ac20--4889--be64--c245859e2063
                                                                        252:1    0 1024G  0 lvm  
vdc                                                                     253:32   0    1T  0 disk 
└─ceph--6c1ebfcb--45a2--4f11--832b--bd9bb15ebedc-osd--block--ae3b152e--dc18--412f--b5fc--775ca8bdb738
                                                                        252:0    0 1024G  0 lvm  
                                                                        
                                                                        
# dmsetup remove ceph--587b3b32--c81a--49ed--8a94--cb7ca9617efb-osd--block--d11ea64c--ac20--4889--be64--c245859e2063   #  ceph-deploy osd create --data /dev/vdb ceph-1                                                                     
```





Q3:添加磁盘报

```bash
#  ceph-deploy osd create --data /dev/vdc ceph-1
... ...
[ceph-1][WARNIN] 2021-09-08 15:39:11.830 7f82bb666700 -1 AuthRegistry(0x7f82b40662f8) no keyring found at /etc/ceph/ceph.client.bootstrap-osd.keyring,/etc/ceph/ceph.keyring,/etc/ceph/keyring,/etc/ceph/keyring.bin,, disabling cephx
[ceph-1][WARNIN]  stderr: purged osd.1
[ceph-1][WARNIN] -->  RuntimeError: command returned non-zero exit status: 5
[ceph-1][ERROR ] RuntimeError: command returned non-zero exit status: 1
```



A3:解决办法

```bash
# dd 清除磁盘头
# dd if=/dev/zero of=/dev/vdc bs=1M count=100
```







Q4:如何查看ceph磁盘信息



A4:

```bash
# ceph -s
  cluster:
    id:     ffafa9e9-4830-47f7-a45f-4a4fe61b2041
    health: HEALTH_WARN
            mons are allowing insecure global_id reclaim
 
  services:
    mon: 3 daemons, quorum ceph-1,ceph-2,ceph-3 (age 4h)
    mgr: ceph-1(active, since 4h), standbys: ceph-2, ceph-3
    mds:  3 up:standby
    osd: 6 osds: 6 up (since 7m), 6 in (since 7m)
    rgw: 1 daemon active (ceph-1)
 
  task status:
 
  data:
    pools:   4 pools, 128 pgs
    objects: 187 objects, 1.2 KiB
    usage:   6.1 GiB used, 6.0 TiB / 6.0 TiB avail
    pgs:     128 active+clean
 
[root@ceph-1 my-cluster]# ceph osd tree
ID CLASS WEIGHT  TYPE NAME       STATUS REWEIGHT PRI-AFF 
-1       6.00000 root default                            
-3       2.00000     host ceph-1                         
 0   hdd 1.00000         osd.0       up  1.00000 1.00000 
 1   hdd 1.00000         osd.1       up  1.00000 1.00000 
-5       2.00000     host ceph-2                         
 2   hdd 1.00000         osd.2       up  1.00000 1.00000 
 3   hdd 1.00000         osd.3       up  1.00000 1.00000 
-7       2.00000     host ceph-3                         
 4   hdd 1.00000         osd.4       up  1.00000 1.00000 
 5   hdd 1.00000         osd.5       up  1.00000 1.00000 
```

