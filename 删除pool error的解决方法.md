删除pool error的解决方法

问题描述：
删除pool的时候提示下面的错误：

```bash
# ceph osd pool delete ecpool ecpool --yes-i-really-really-mean-it
Error EPERM: pool deletion is disabled; you must first set the mon_allow_pool_delete config option to true before you can destroy a pool
```



这是由于没有配置mon节点的 mon_allow_pool_delete 字段所致，解决办法就是到mon节点进行相应的设置

解决方法：
注：1-3步的操作必须在mon节点上执行

1. 打开mon节点的配置文件：

  ```bash
  [root@node1 ceph]# vi /etc/ceph/ceph.conf 
  ```

  

2. 在配置文件中添加如下内容：

  ```bash
  [mon]
  mon allow pool delete = true
  ```

  

3. 重启ceph-mon服务：
```bash
# systemctl restart ceph-mon.target
```



4. 执行删除pool命令：

  ```bash
  [root@node3 ~]# ceph osd pool delete ecpool ecpool --yes-i-really-really-mean-it
  pool 'ecpool' removed
  ```

  