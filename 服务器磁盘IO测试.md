sda随机写IOPS
```bash
fio -direct=1 -iodepth=128 -rw=randwrite -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Rand_Write_Testing
```



write: IOPS=52.3k, BW=204MiB/s (214MB/s)(1024MiB/5014msec)

sdb随机写IOPS
```bash
fio -direct=1 -iodepth=128 -rw=randwrite -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Rand_Write_Testing
```



write: IOPS=53.0k, BW=207MiB/s (217MB/s)(1024MiB/4942msec)

sdc随机写IOPS
```bash
fio -direct=1 -iodepth=128 -rw=randwrite -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Rand_Write_Testing
```



write: IOPS=48.8k, BW=191MiB/s (200MB/s)(1024MiB/5372msec)




sda顺序写IOPS
```bash
fio -direct=1 -iodepth=128 -rw=write -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Rand_Write_Testing
```



write: IOPS=90.4k, BW=353MiB/s (370MB/s)(1024MiB/2900msec)

sdb顺序写IOPS
```bash
fio -direct=1 -iodepth=128 -rw=write -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Rand_Write_Testing
```



write: IOPS=90.0k, BW=355MiB/s (373MB/s)(1024MiB/2881msec)

sdc顺序写IOPS
```bash
fio -direct=1 -iodepth=128 -rw=write -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Rand_Write_Testing
```



write: IOPS=91.6k, BW=358MiB/s (375MB/s)(1024MiB/2863msec)






sda随机读IOPS
```bash
fio -direct=1 -iodepth=128 -rw=randread -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Rand_Read_Testing
```



read: IOPS=80.1k, BW=313MiB/s (328MB/s)(1024MiB/3273msec)

sdb随机读IOPS
```bash
fio -direct=1 -iodepth=128 -rw=randread -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Rand_Read_Testing
```



read: IOPS=81.0k, BW=317MiB/s (332MB/s)(1024MiB/3235msec)

sdc随机读IOPS
```bash
fio -direct=1 -iodepth=128 -rw=randread -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Rand_Read_Testing
```



read: IOPS=80.4k, BW=314MiB/s (329MB/s)(1024MiB/3260msec)



sda顺序读IOPS
```bash
fio -direct=1 -iodepth=128 -rw=read -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Rand_Read_Testing
```



read: IOPS=116k, BW=452MiB/s (474MB/s)(1024MiB/2263msec)

sdb顺序读IOPS
```bash
fio -direct=1 -iodepth=128 -rw=read -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Rand_Read_Testing
```



read: IOPS=115k, BW=450MiB/s (472MB/s)(1024MiB/2276msec)

sdc顺序读IOPS
```bash
fio -direct=1 -iodepth=128 -rw=read -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Rand_Read_Testing
```



read: IOPS=114k, BW=447MiB/s (468MB/s)(1024MiB/2293msec)





  

sda顺序写吞吐量（写带宽）
```bash
fio -direct=1 -iodepth=64 -rw=write -ioengine=libaio -bs=1024k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Write_PPS_Testing
```



write: IOPS=494, BW=495MiB/s (519MB/s)(1024MiB/2070msec)

sdb顺序写吞吐量（写带宽）
```bash
fio -direct=1 -iodepth=64 -rw=write -ioengine=libaio -bs=1024k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Write_PPS_Testing
```



write: IOPS=495, BW=496MiB/s (520MB/s)(1024MiB/2066msec)

sdc顺序写吞吐量（写带宽）
```bash
fio -direct=1 -iodepth=64 -rw=write -ioengine=libaio -bs=1024k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Write_PPS_Testing
```



write: IOPS=496, BW=496MiB/s (520MB/s)(1024MiB/2064msec)



sda顺序读吞吐量（读带宽）  
```bash
fio -direct=1 -iodepth=64 -rw=read -ioengine=libaio -bs=1024k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Read_PPS_Testing
```



read: IOPS=533, BW=534MiB/s (560MB/s)(1024MiB/1918msec)

sdb顺序读吞吐量（读带宽）  
```bash
fio -direct=1 -iodepth=64 -rw=read -ioengine=libaio -bs=1024k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Read_PPS_Testing
```



read: IOPS=533, BW=534MiB/s (560MB/s)(1024MiB/1918msec)

sdc顺序读吞吐量（读带宽）  
```bash
fio -direct=1 -iodepth=64 -rw=read -ioengine=libaio -bs=1024k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Read_PPS_Testing
```



read: IOPS=534, BW=534MiB/s (560MB/s)(1024MiB/1916msec)




sda随机写时延
```bash
fio -direct=1 -iodepth=1 -rw=randwrite -ioengine=libaio -bs=4k -size=1G -numjobs=1 -group_reporting -filename=iotest -name=Rand_Write_Latency_Testing
```



write: IOPS=20.4k, BW=79.9MiB/s (83.7MB/s)(1024MiB/12823msec)

sdb随机写时延
```bash
fio -direct=1 -iodepth=1 -rw=randwrite -ioengine=libaio -bs=4k -size=1G -numjobs=1 -group_reporting -filename=iotest -name=Rand_Write_Latency_Testing
```



write: IOPS=20.0k, BW=78.1MiB/s (81.9MB/s)(1024MiB/13105msec)

sdc随机写时延
```bash
fio -direct=1 -iodepth=1 -rw=randwrite -ioengine=libaio -bs=4k -size=1G -numjobs=1 -group_reporting -filename=iotest -name=Rand_Write_Latency_Testing
```



write: IOPS=17.6k, BW=68.8MiB/s (72.1MB/s)(1024MiB/14886msec)




sda随机读时延  
```bash
fio -direct=1 -iodepth=1 -rw=randread -ioengine=libaio -bs=4k -size=1G -numjobs=1 -group_reporting -filename=iotest -name=Rand_Read_Latency_Testing
```



read: IOPS=12.4k, BW=48.5MiB/s (50.9MB/s)(1024MiB/21105msec)

sdb随机读时延  
```bash
fio -direct=1 -iodepth=1 -rw=randread -ioengine=libaio -bs=4k -size=1G -numjobs=1 -group_reporting -filename=iotest -name=Rand_Read_Latency_Testing
```



read: IOPS=12.4k, BW=48.5MiB/s (50.9MB/s)(1024MiB/21093msec)

sdc随机读时延  
```bash
fio -direct=1 -iodepth=1 -rw=randread -ioengine=libaio -bs=4k -size=1G -numjobs=1 -group_reporting -filename=iotest -name=Rand_Read_Latency_Testing
```



read: IOPS=12.4k, BW=48.6MiB/s (50.9MB/s)(1024MiB/21081msec)