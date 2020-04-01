## 一、说明

在K8S集群中，通过ceph-csi使用ceph的块设备镜像方式，动态提供RBD镜像给K8S卷使用

![微信截图_20200331095006.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gdcvep4vfbj30f706u0sr.jpg)



## 二、ceph配置静态PV

2.1、在node1上配置pool

```bash
[root@node1 ceph]# ceph osd pool create kubernetes 256
pool 'kubernetes' created

[root@node1 ceph]# ceph osd lspools
1 data_data,2 data_metadata,4 kubernetes,
```



2.2、在pool中创建rbd image

```bash
[root@node1 ceph]# rbd create kube --size 64 -p kubernetes

#查看信息
[root@node1 ceph]# rbd info kube -p kubernetes
rbd image 'kube':
        size 64MiB in 16 objects
        order 22 (4MiB objects)
        block_name_prefix: rbd_data.108b6b8b4567
        format: 2
        features: layering, exclusive-lock, object-map, fast-diff, deep-flatten
        flags: 
        create_timestamp: Tue Mar 31 11:16:30 2020
```



2.3、移除image的权限

```bash
[root@node1 ceph]# rbd feature disable kube -p kubernetes object-map fast-diff deep-flatten

#查看
[root@node1 ceph]# rbd info kube -p kubernetes
rbd image 'kube':
        size 64MiB in 16 objects
        order 22 (4MiB objects)
        block_name_prefix: rbd_data.108b6b8b4567
        format: 2
        features: layering, exclusive-lock
        flags: 
        create_timestamp: Tue Mar 31 11:16:30 2020
```





## 三、K8S配置 

备注：以下配置是在k8s的master节点上进行



3.1、配置yum

```bash
[root@k8s-master yum.repos.d]# cat ceph.repo 
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


[root@k8s-master yum.repos.d]# yum makecache
```



3.2、安装ceph-common

```bash
[root@k8s-master yum.repos.d]#  yum -y install ceph-common
```



3.3、从ceph节点拷贝文件到k8s master下

```bash
[root@k8s-master yum.repos.d]# scp 192.168.255.196:/etc/ceph/ceph.client.admin.keyring /etc/ceph/
[root@k8s-master yum.repos.d]# scp 192.168.255.196:/etc/ceph/ceph.conf /etc/ceph/       
```



3.4、从k8s master上检查是否能看到ceph pool信息

```bash
[root@k8s-master yum.repos.d]# rados lspools    #有pool信息
data_data
data_metadata
kubernetes
```



3.5、获取client.admin的keyring值，并用base64编码

```bash
[root@k8s-master ceph]#  ceph auth get-key client.admin | base64  
QVFDeGpZRmVMMWRHT1JBQVlldGJxamljcEwrNDNtM3BPd1BMQWc9PQ==
```



3.6、配置ceph secret

```bash
[root@k8s-master K8s-Ceph]# cat ceph-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: ceph-secret
type: kubernetes.io/rbd
data:
  key: QVFDeGpZRmVMMWRHT1JBQVlldGJxamljcEwrNDNtM3BPd1BMQWc9PQ==
  
  
[root@k8s-master K8s-Ceph]# kubectl create -f ceph-secret.yaml 
secret/ceph-secret created


[root@k8s-master K8s-Ceph]#  kubectl get secret ceph-secret
NAME          TYPE                DATA   AGE
ceph-secret   kubernetes.io/rbd   1      18s
```



3.7、创建PV 

```bash
[root@k8s-master K8s-Ceph]# vim ceph-pv.yaml 

apiVersion: v1
kind: PersistentVolume
metadata:
  name: ceph-pv
spec:
  capacity:
    storage: 50Mi
  accessModes:
    - ReadWriteOnce
  storageClassName: "rbd"
  rbd:
    monitors:
      - 192.168.255.196:6789
      - 192.168.255.197:6789
      - 192.168.255.198:6789
    pool: kubernetes    #pool名称
    image: kube         #image名称
    user: admin
    secretRef:
      name: ceph-secret
    fsType: xfs
    readOnly: false
  persistentVolumeReclaimPolicy: Recycle
  
[root@k8s-master K8s-Ceph]# kubectl create -f ceph-pv.yaml 

# kubectl get pv
NAME            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                    STORAGECLASS   REASON   AGE
ceph-pv         50Mi       RWO            Recycle          Available                            rbd                     21s

```



3.8、创建PVC

```bash
[root@k8s-master K8s-Ceph]# vim ceph-claim.yaml 
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ceph-claim
spec:
  storageClassName: "rbd"
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Mi
	  
[root@k8s-master K8s-Ceph]# kubectl create -f ceph-claim.yaml 

[root@k8s-master K8s-Ceph]# kubectl get pvc
NAME                 STATUS    VOLUME          CAPACITY   ACCESS MODES   STORAGECLASS   AGE
ceph-claim           Bound     ceph-pv         50Mi       RWO            rbd            17s


[root@k8s-master K8s-Ceph]#  kubectl get pv
NAME            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS   REASON   AGE
ceph-pv         50Mi       RWO            Recycle          Bound    default/ceph-claim       rbd                     4m4s
```



3.9、创建pod

```bash
[root@k8s-master K8s-Ceph]# vim ceph-pod1.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: ceph-pod1           
spec:
  containers:
  - name: ceph-busybox
    image: busybox          
    command: ["sleep", "60000"]
    volumeMounts:
    - name: ceph-vol1       
      mountPath: /usr/share/busybox 
      readOnly: false
  volumes:
  - name: ceph-vol1         
    persistentVolumeClaim:
      claimName: ceph-claim
	  
[root@k8s-master K8s-Ceph]# kubectl get pod ceph-pod1 -o wide
NAME        READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
ceph-pod1   1/1     Running   0          75s   10.244.0.104   k8s-master   <none>           <none>
```



3.10、测试

进入到该Pod中，向/usr/share/busybox目录写入一些数据，之后删除该Pod，再创建一个新的Pod,看之前的数据是否还存在。

```bash
[root@k8s-master K8s-Ceph]# kubectl exec -it ceph-pod1 -- /bin/sh
/ # ls
bin   dev   etc   home  proc  root  sys   tmp   usr   var
/ # cd /usr/share/busybox
/usr/share/busybox # ls
/usr/share/busybox #  echo 'Hello from Kubernetes storage' > k8s.txt
/usr/share/busybox # cat k8s.txt 
Hello from Kubernetes storage
/usr/share/busybox # exit
[root@k8s-master K8s-Ceph]# kubectl delete pod ceph-pod1
pod "ceph-pod1" deleted
[root@k8s-master K8s-Ceph]# kubectl apply -f ceph-pod1.yaml 
pod/ceph-pod1 created

[root@k8s-master K8s-Ceph]# kubectl exec ceph-pod1 -- cat /usr/share/busybox/k8s.txt #数据依然存在
Hello from Kubernetes storage
```





## 四、配置动态PV

4.1、创建RBD pool

```bash
[root@node1 yum.repos.d]# ceph osd pool create kube 128
pool 'kube' created
```



4.2、授权kube用户

```bash
[root@node1 yum.repos.d]# ceph auth get-or-create client.kube mon 'allow r' osd 'allow class-read,allow rwx pool=kube' -o  ceph.client.kube.keyring

[root@node1 yum.repos.d]#  ceph auth get client.kube
exported keyring for client.kube
[client.kube]
        key = AQAHKINeJeI2JhAAq1l7D8a81BYLX6DDlTRxyg==
        caps mon = "allow r"
        caps osd = "allow class-read,allow rwx pool=kube"
```



备注：

Ceph使用术语“capabilities”(caps)来描述授权经过身份验证的用户使用监视器、OSD和元数据服务器的功能。功能还可以根据应用程序标记限制对池中的数据，池中的命名空间或一组池的访问。Ceph管理用户在创建或更新用户时设置用户的功能。

Mon 权限: 包括 r 、 w 、 x 。
OSD 权限: 包括 r 、 w 、 x 、 class-read 、 class-write 。另外，还支持存储池和命名空间的配置



4.3、创建ceph secret

```bash


[root@k8s-master K8s-Ceph]#  ceph auth get-key client.admin | base64     #获取client.admin的keyring值，并用base64编码

QVFDeGpZRmVMMWRHT1JBQVlldGJxamljcEwrNDNtM3BPd1BMQWc9PQ==

[root@k8s-master K8s-Ceph]# ceph auth get-key client.kube|base64         #获取client.admin的keyring值，并用base64编码
QVFBSEtJTmVKZUkySmhBQXExbDdEOGE4MUJZTFg2RERsVFJ4eWc9PQ==


[root@k8s-master K8s-Ceph]# cat ceph-kube-secret.yaml 
apiVersion: v1
kind: Namespace
metadata:
  name: ceph
---
apiVersion: v1
kind: Secret
metadata:
  name: ceph-admin-secret
  namespace: ceph
type: kubernetes.io/rbd
data:
  key: QVFDeGpZRmVMMWRHT1JBQVlldGJxamljcEwrNDNtM3BPd1BMQWc9PQ==
---
apiVersion: v1
kind: Secret
metadata:
  name: ceph-kube-secret
  namespace: ceph
type: kubernetes.io/rbd
data:
  key: QVFBSEtJTmVKZUkySmhBQXExbDdEOGE4MUJZTFg2RERsVFJ4eWc9PQ==
  
  
  [root@k8s-master K8s-Ceph]# kubectl create -f ceph-kube-secret.yaml 
namespace/ceph created
secret/ceph-admin-secret created
secret/ceph-kube-secret created


[root@k8s-master K8s-Ceph]# kubectl get secret -n ceph
NAME                  TYPE                                  DATA   AGE
ceph-admin-secret     kubernetes.io/rbd                     1      26s
ceph-kube-secret      kubernetes.io/rbd                     1      25s
default-token-klc8c   kubernetes.io/service-account-token   3      25s
```



4.4、创建动态RBD StorageClass

```bash
[root@k8s-master K8s-Ceph]# cat ceph-storageclass.yaml 
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ceph-rbd
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: kubernetes.io/rbd
parameters:
  monitors: 192.168.255.196:6789,192.168.255.197:6789,192.168.255.198:6789
  adminId: admin
  adminSecretName: ceph-admin-secret
  adminSecretNamespace: ceph
  pool: kube
  userId: kube
  userSecretName: ceph-kube-secret
  fsType: xfs
  imageFormat: "2"
  imageFeatures: "layering"
  
  [root@k8s-master K8s-Ceph]# kubectl create -f ceph-storageclass.yaml 
storageclass.storage.k8s.io/ceph-rbd created


[root@k8s-master K8s-Ceph]# kubectl get sc
NAME                 PROVISIONER         AGE
ceph-rbd (default)   kubernetes.io/rbd   19s
```

备注:  
storageclass.kubernetes.io/is-default-class：注释为true，标记为默认的StorageClass，注释的任何其他值或缺失都被解释为false。
monitors：Ceph监视器，逗号分隔。此参数必需。
adminId：Ceph客户端ID，能够在pool中创建images。默认为“admin”。
adminSecretNamespace：adminSecret的namespace。默认为“default”。
adminSecret：adminId的secret。此参数必需。提供的secret必须具有“kubernetes.io/rbd”类型。
pool：Ceph RBD池。默认为“rbd”。
userId：Ceph客户端ID，用于映射RBD image。默认值与adminId相同。
userSecretName：用于userId映射RBD image的Ceph Secret的名称。它必须与PVC存在于同一namespace中。此参数必需。提供的secret必须具有“kubernetes.io/rbd”类型，例如以这种方式创建： 
kubectl create secret generic ceph-secret --type="kubernetes.io/rbd" \ --from-literal=key='QVFEQ1pMdFhPUnQrSmhBQUFYaERWNHJsZ3BsMmNjcDR6RFZST0E9PQ==' \ --namespace=kube-system
fsType：kubernetes支持的fsType。默认值："ext4"。
imageFormat：Ceph RBD image格式，“1”或“2”。默认值为“1”。
imageFeatures：此参数是可选的，只有在设置imageFormat为“2”时才能使用。目前仅支持的功能为layering。默认为“”，并且未开启任何功能。
默认的StorageClass标记为(default)  





4.5、创建PVC

通过在其中包含存储类来请求动态调配存储PersistentVolumeClaim

```bash
[root@k8s-master K8s-Ceph]# cat ceph-pvc.yaml 
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ceph-pvc
  namespace: ceph
spec:
  storageClassName: ceph-rbd
  accessModes:
     - ReadOnlyMany
  resources:
    requests:
      storage: 1Gi
      
 
 [root@k8s-master K8s-Ceph]# kubectl create -f ceph-pvc.yaml 
persistentvolumeclaim/ceph-pvc created

[root@k8s-master K8s-Ceph]# kubectl get pvc -n ceph
```



4.6、创建pod并测试 

```bash
# vim ceph-pod2.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: ceph-pod2
  namespace: ceph
spec:
  containers:
  - name: ceph-busybox
    image: busybox
    command: ["sleep", "60000"]
    volumeMounts:
    - name: ceph-vol1
      mountPath: /usr/share/busybox
      readOnly: false
  volumes:
  - name: ceph-vol1
    persistentVolumeClaim:
      claimName: ceph-pvc
  
# kubectl create -f ceph-pod2.yaml 
pod/ceph-pod2 created
# kubectl -n ceph get pod ceph-pod2 -o wide
NAME        READY   STATUS   RESTARTS   AGE   IP           NODE
ceph-pod2   1/1     Running  0          3m    10.244.88.2  node03
# kubectl -n ceph exec -it ceph-pod2 -- /bin/sh
/ # echo 'Ceph from Kubernetes storage' > /usr/share/busybox/ceph.txt
/ # exit
# kubectl -n ceph delete pod ceph-pod2
pod "ceph-pod2" deleted
# kubectl apply -f ceph-pod2.yaml
pod/ceph-pod2 created
# kubectl -n ceph get pod ceph-pod2 -o wide
NAME        READY     STATUS    RESTARTS   AGE  IP            NODE
ceph-pod2   1/1       Running   0          2m   10.244.88.2   node03
# kubectl -n ceph exec ceph-pod2 -- cat /usr/share/busybox/ceph.txt
Ceph from Kubernetes storage
```

