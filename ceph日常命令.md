1、查看ceph存储池

```bash
# ceph osd lspools
1 .rgw.root
2 default.rgw.control
3 default.rgw.meta
4 default.rgw.log
5 rbd
6 pool_mysh
```



2、查看存储池副本数

```bash
# ceph osd pool get pool_mysh size
size: 3
```



3、删除存储池

```bash
# ceph osd pool delete pool_mysh pool_mysh --yes-i-really-really-mean-it
```

如果报

```bash
Error EPERM: pool deletion is disabled; you must first set the mon_allow_pool_delete config option to true before you can destroy a pool
```

是由于没有配置mon节点的 mon_allow_pool_delete 字段所致，解决办法就是到mon节点进行相应的设置



解决办法：

a)、配置mon节点文件

```bash
# vi /etc/ceph/ceph.conf
# 添加
[mon]
mon_allow_pool_delete = true
                      
```

b)、推送配置

```bash
# cd /etc/ceph
# ceph-deploy --overwrite-conf config push ceph1 ceph2 ceph3
```





c)、重启mon服务

```bash
# for i in {1..3}
> do ssh ceph${i} "\
> systemctl restart ceph-mon.target
> "
> done
```



d)、再执行删除

```bash
#  ceph osd pool delete pool_mysh pool_mysh --yes-i-really-really-mean-it
pool 'pool_mysh' removed
```





4、创建存储池 

通常在创建pool之前，需要覆盖默认的pg_num，官方推荐：

若少于5个OSD， 设置pg_num为128。

5~10个OSD，设置pg_num为512。

10~50个OSD，设置pg_num为4096。

超过50个OSD，可以参考pgcalc计算。

```bash
# ceph osd pool create  pool_mysh 120
```



5、修改存储池副本数

```bash
# ceph osd pool set pool_mysh size 1
set pool 7 size to 1

#  ceph osd pool get pool_mysh size 
size: 1
```



6、获取池pg_num/pgp_num

```bash
# ceph osd pool get pool_mysh  pg_num
pg_num: 120

# ceph osd pool get pool_mysh  pgp_num
pgp_num: 120
```



7、设置池pg_num/pgp_num

```bash
# ceph osd pool set pool_mysh pg_num 128
set pool 7 pg_num to 128

# ceph osd pool set pool_mysh pgp_num 128
set pool 7 pgp_num to 128
```



8、查看存储池统计信息

```bash
# rados df
POOL_NAME              USED OBJECTS CLONES COPIES MISSING_ON_PRIMARY UNFOUND DEGRADED RD_OPS      RD WR_OPS    WR USED COMPR UNDER COMPR 
.rgw.root           768 KiB       4      0     12                  0       0        0      0     0 B      4 4 KiB        0 B         0 B 
default.rgw.control     0 B       8      0     24                  0       0        0      0     0 B      0   0 B        0 B         0 B 
default.rgw.log         0 B     207      0    621                  0       0        0 124454 121 MiB  82930   0 B        0 B         0 B 
default.rgw.meta        0 B       0      0      0                  0       0        0      0     0 B      0   0 B        0 B         0 B 
pool_mysh               0 B       0      0      0                  0       0        0      0     0 B      0   0 B        0 B         0 B 

total_objects    219
total_used       3.0 GiB
total_avail      297 GiB
total_space      300 GiB
```



9、查看存储池配额

```bash
# ceph osd pool get-quota pool_mysh
quotas for pool 'pool_mysh':
  max objects: N/A
  max bytes  : N/A
```

