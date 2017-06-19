# <font color="red">Storm</font>持续集成部署平台


## 平台建设目标

> 持续集成 持续部署   

> 敏捷开发的基础支持平台

> 自动化软件生命周期流转

> 提升运维人员工作效率，提升价值的利器

## 平台组件

 * Subversion / Gitlab
 * Jenkins
 * Sonarqube
 * Kubernetes
 * Docker

## 平台架构及拓扑

## 各组件安装

### Jenkins安装

Jenkins 将以容器安装的方式部署在Kubernetes平台上，以最简化安装过程以及后续维护。后台存储选用Ceph分布式存储，以利用其多副本等高可靠以及灵活扩展的特性。

安装过程如下：

1. 持久化存储创建
   登陆Ceph服务器，然后执行下述指令

   ```shell
   rbd create rbd/jenkins-home --size 20G --image-format 2 --image-feature layering
   ```
2. Kubernetes PV & PVC 创建

   在kubernetes Dashboard以图形化界面方式使用下述文件创建或者上传文件到Kubernetes Master后执行
```
kubectl create -f jenkins-storage.yaml

```

   jenkins-storage.yaml

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: jenkins-home-pv
spec:
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  rbd:
    monitors:
      - 10.72.243.113:6789,10.72.243.114:6789,10.72.243.115:6789
    pool: rbd
    image: jenkins-home
    user: admin
    secretRef:
      name: ceph-secret
    fsType: ext4
    readOnly: false
  persistentVolumeReclaimPolicy: Recycle
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: jenkins-home-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
```

3. 在kubernetes平台上创建Jenkins

***注意：***
***Jenkins Base Image Dockerfile***

(该文件尚待修改，目前使用当前使用的定制版本)
```Dockerfile
FROM docker.io/library/Jenkins

ADD  ORALCE JAVA (选那个JDK需要考虑)
ADD  MYSQL (为未来执行Mysql Script)
ADD  Maven

```

Jenins.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: jenkins-research-svc
  labels:
    name: jenkins
spec:
  type: LoadBalancer
  selector:
    name: jenkins-research
  ports:
    - name: http
      port: 8080

---     
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: jenkins-research-deployment
spec:
  replicas: 1
  template:
    metadata:
      labels:
        name: jenkins-research
    spec:
      containers:
      - name: jenkins-research
        image: registry.aegonthtf.com/aegonthtf-research/jenkins-research
        imagePullPolicy: Always       
        ports:
        - name: http
          containerPort: 8080
        volumeMounts:
        - name: jenkins-home
          mountPath: /var/jenkins_home
      volumes:
      - name: jenkins-home
        persistentVolumeClaim:
          claimName: jenkins-home-claim

```

> 图 - K8s 上的Jenkins svc 示意图

### Jenkins配置


* Unlock
* Proxy Setting
* Install Plugins
* Create Admin Accounts
* General Setting
*

### Jenkins Pipeline

### Use Case
