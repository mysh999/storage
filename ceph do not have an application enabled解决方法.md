Q: ceph集群执行ceph -s时，报以下提示

```bash
# ceph -s
  cluster:
    id:     dc521459-398b-4f8f-b86c-dac03dc060b4
    health: HEALTH_WARN
            1 pool(s) do not have an application enabled
 
  services:
    mon: 3 daemons, quorum ceph-027,ceph-028,ceph-029 (age 5d)
    mgr: ceph-027(active, since 5d), standbys: ceph-029, ceph-028
    mds: fs01:1 {0=ceph-029=up:active} 2 up:standby
    osd: 6 osds: 6 up (since 5d), 6 in (since 5d)
    rgw: 3 daemons active (ceph-027, ceph-028, ceph-029)
 
  task status:
    scrub status:
        mds.ceph-029: idle
 
  data:
    pools:   8 pools, 201 pgs
    objects: 255 objects, 55 KiB
    usage:   7.6 GiB used, 6.0 TiB / 6.0 TiB avail
    pgs:     201 active+clean
```

其中

1 pool(s) do not have an application enabled



解决办法：

```bash
# 查看详细信息
# ceph health detail
HEALTH_WARN 1 pool(s) do not have an application enabled
[WRN] POOL_APP_NOT_ENABLED: 1 pool(s) do not have an application enabled
    application not enabled on pool 'cephrbd'
    use 'ceph osd pool application enable <pool-name> <app-name>', where <app-name> is 'cephfs', 'rbd', 'rgw', or freeform for custom applications.
    
 #配置RBD类型
[root@ceph-027 ~]# ceph osd pool application enable cephrbd rbd
enabled application 'rbd' on pool 'cephrbd'

# 再次查看，无告警
[root@ceph-027 ~]# ceph -s
  cluster:
    id:     dc521459-398b-4f8f-b86c-dac03dc060b4
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum ceph-027,ceph-028,ceph-029 (age 5d)
    mgr: ceph-027(active, since 5d), standbys: ceph-029, ceph-028
    mds: fs01:1 {0=ceph-029=up:active} 2 up:standby
    osd: 6 osds: 6 up (since 5d), 6 in (since 5d)
    rgw: 3 daemons active (ceph-027, ceph-028, ceph-029)
 
  task status:
    scrub status:
        mds.ceph-029: idle
 
  data:
    pools:   8 pools, 201 pgs
    objects: 255 objects, 55 KiB
    usage:   7.6 GiB used, 6.0 TiB / 6.0 TiB avail
    pgs:     201 active+clean
```



