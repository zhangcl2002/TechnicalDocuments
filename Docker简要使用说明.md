> # 本文目标

本文希望做一个向导，对Docker/K8S最通用的使用场景和过程做一个说明。 希望阅读本文对如何在我们的容器云平台上部署一个应用软件能起到一个手边参考的作用。

阅读本文需要对k8s/docker的概念有一个基础的了解,也可以在阅读过程中对不明白的概念进行查询了解。

本文拟按顺序描述部署一个应用软件的过程。 此处需要说明的是，在真实的实践中，我们会对业务应用的部署封装大部分手工操作，利用jenkins实现自动部署，但运维使用的一些非业务应用的系统软件可能大部分情况下还是会采用手工部署的形式。 但无论那种形式，基础步骤都是一致差不多的。

本文分如下几节：

- 镜像构建

  - 了解基础镜像
  - 编写Dockerfile
  - 生成镜像并推送至镜像仓库

- 持久存储配备(optional)

  - 在持久存储上创建需要的存储位置
  - 在K8s上创建PV, PVC

- 在k8s上部署服务

  - 通过k8s Dashboard界面部署服务
  - 通过yaml文件部署服务

- 服务验证

  > # 主要步骤

> ## 构建应用容器镜像

1. 了解基础镜像 Docker的镜像是分层的，一层一层叠加的。比如，最底层就是scratch, 意思就是什么也没有，然后上面可以是CentOS, 然后上面是Tomcat,然后再上面是将实际的业务应用以及对Tomcat的定制化内容加入，最终形成一个实际可以使用的应用系统的Docker镜像。<br>

2. 编写Dockerfile文件

Dockerfile是生成Docker镜像的脚本。

比如CentOS： <https://github.com/CentOS/sig-cloud-instance-images/blob/5bbaef9f60ab9e3eeb61acec631c2d91f8714fff/docker/Dockerfile>

比如Tomcat： <https://github.com/docker-library/tomcat/blob/e45349ec3432c75ca5b7e2ef4cae95276c7dccc7/8.0/jre8/Dockerfile>

生成镜像并推送至镜像仓库 将所有相关的文件放在安装有Docker Engine的服务器上某个文件夹下，然后在该文件夹执行下面的命令

Dockerfile:

```
FROM registry.aegonthtf.com/aegonthtf-research/tomcat:8.0-jre8

MAINTAINER "Michael Zhang" <michaelzhang@aegonthtf.com>
ENV LANG en_US.utf8
ENV JAVA_OPTS '-server -Dfile.encoding=UTF-8 -Duser.timezone=Asia/Shanghai'

#如果选用类Spring的方式，不采用上下文，直接进入应用的话可采用如下.( 注意要注释掉传统方式那条)
ADD target/tra-app.war /usr/local/tomcat/webapps/ROOT.war

#传统方式
#ADD target/tra-app.war /usr/local/tomcat/webapps/tra-app.war
```

1. 生成镜像并保存至私有仓库

执行如下命令，生成tra-app应用镜像

```
docker build -t registry.aegonthtf.com/aegonthtf-research/tra-app:23 .
```

推送至私有仓库

```
docker push registry.aegonthtf.com/aegonthtf-research/tra-app:23
```

<br><br>

> ## 持久存储准备

容器可以使用多种持久存储，比如本地硬盘，NAS共享文件存储，Ceph块存储，Ceph文件存储,NFS共享文件存储。 此文以我们可能会使用的NAS和Ceph举例。

容器如不使用持久存储，则当容器重启的时候，运行时数据将全部丢失，重启后得到一个全新的实例。

## 1. 在持久存储上创建需要的存储位置

### Ceph 存储

- 创建ceph 块存储image 登陆可以操作ceph的机器，创建ceph image (这是简要的使用，实际更完善的使用访问以后再逐步完善)

  ```bash
  rbd create rbd/demo-storage --size 5G --image-format 2 --image-feature layering
  ```

- 查看

  ```bash
  rbd ls
  rbd info demo-storage
  ```
下面几个命令参考，不要执行
  - 删除镜像

  ```bash
  rbd rm demo-storage
  ```
  - 查看锁
  ```bash
[root@vm-shalce97 ~]# rbd lock list demo-storage
There is 1 exclusive lock on this image.
Locker         ID                                           Address
client.2265893 kubelet_lock_magic_vm-shalku93.aegonthtf.com 10.72.243.139:0/1022248

  ```
  - 删除锁
  ```bash
[root@vm-shalce97 ~]# rbd lock rm demo-storage kubelet_lock_magic_vm-shalku93.aegonthtf.com client.2265893
  ```

  - 扩展image大小
  ```bash
  [root@vm-shalce97 ~]# rbd resize -p rbd --size 10G demo-storage
Resizing image: 100% complete...done.
[root@vm-shalce97 ~]# rbd info demo-storage
rbd image 'demo-storage':
	size 10240 MB in 2560 objects
	order 22 (4096 kB objects)
	block_name_prefix: rbd_data.2292dd6b8b4567
	format: 2
	features: layering
	flags:
[root@vm-shalce97 ~]#
  ```


演示： ![ceph image create](https://mmbiz.qlogo.cn/mmbiz_gif/5Ofd65QfQBDrn9XqAib8CAY8S3Xty6ibDlIe9KriaujysUBzXmBR4Eso9ROzUO3VW9vMo2F3JDwD7qx1Z7L123pKw/0?wx_fmt=gif)

- 在k8s中创建PV & PVC

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: demo-ceph-pv
  labels:
    type: ceph-demo
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  rbd:
    monitors:
      - 10.72.243.113:6789,10.72.243.114:6789,10.72.243.115:6789
    pool: rbd
    image: demo-storage
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
  name: demo-ceph-pvc
  labels:
    type: ceph-demo
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

演示： ![ceph image create](https://mmbiz.qlogo.cn/mmbiz_png/5Ofd65QfQBDrn9XqAib8CAY8S3Xty6ibDltRZAqfWribLC0m6UlYgnia9Yc57Zl5AKPyLxGRJB8xLkMd1XmmakWW8g/0?wx_fmt=png)

### NAS 存储

- 在k8s各Node上映射NAS应用共享文件夹路径

![nas1](https://mmbiz.qlogo.cn/mmbiz_png/5Ofd65QfQBDrn9XqAib8CAY8S3Xty6ibDlw0z1Ar7ic5tgfib1tdNA2lojf3BfrqhglW3FdCJdCZtyM8Q1oojkX3Pw/0?wx_fmt=png)

- 创建PV&PVC

```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: demo-nas-pv
  labels:
    type: local-demo
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/appdata/"
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: demo-nas-pvc
  labels:
    type: local-demo
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
```

演示： ![nas storage create](https://mmbiz.qlogo.cn/mmbiz_gif/5Ofd65QfQBDrn9XqAib8CAY8S3Xty6ibDloXGsRJkbZZ9vkA38kVEk95ibQmWyWzKsIlkCptNRrbEIHPIibkP89NPA/0?wx_fmt=gif)

> ## 在k8s上部署服务

- **通过k8s Dashboard界面部署** 通过k8s dashboard图形界面可以部署非常简单的应用。做学习研究时，仅部署简单应用时可尝试使用。 复杂的应用，比如要有持久存储的，要配置探测器等，需要使用下面的脚本方法。 相信未来图形界面也会有进步会支持复杂特性的直观操作。

- **通过yaml文件部署服务**

使用不同持久存储，在脚本上仅仅是claimName的值不同而已。 可以参见下面两个脚本。

使用Ceph存储的应用部署

```yaml
kind: Service
apiVersion: v1
metadata:
  name: demo-svc
spec:
  ports:
    - name: http
      port: 8080
      targetPort: 8080
  selector:
      app: demo
  type: ClusterIP
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name:  demo-deployment
  namespace: default
  labels:
    app:  demo
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: demo
    spec:
      terminationGracePeriodSeconds: 35
      containers:
      - name: task
        image: registry.aegonthtf.com/aegonthtf-research/tra-app:23
        imagePullPolicy: Always
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 60
          timeoutSeconds: 6
          successThreshold: 1
          failureThreshold: 5
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 60
          timeoutSeconds: 6
          successThreshold: 1
          failureThreshold: 5
        ports:
          - name: http
            containerPort: 8080
        volumeMounts:
              -
                name: "persistent-storage"
                mountPath: "/appdata"
      volumes:
          -
            name: "persistent-storage"
            persistentVolumeClaim:
            claimName: demo-ceph-pvc
```

演示：![app create](https://mmbiz.qlogo.cn/mmbiz_gif/5Ofd65QfQBDrn9XqAib8CAY8S3Xty6ibDl7mnfZmCkhViaqicgq83Tf7KYfDNretpHIFJEI836nnRjKTA7vcCw1n5A/0?wx_fmt=gif)

![result](https://mmbiz.qlogo.cn/mmbiz_png/5Ofd65QfQBDrn9XqAib8CAY8S3Xty6ibDltRZAqfWribLC0m6UlYgnia9Yc57Zl5AKPyLxGRJB8xLkMd1XmmakWW8g/0?wx_fmt=png)

_注意：使用ceph块存储不能创建多于1个副本，否则第二个及以上的副本会遇到存储被锁住的提示_

使用NAS存储的应用部署.

```yaml
kind: Service
apiVersion: v1
metadata:
  name: demo-svc
spec:
  ports:
    - name: http
      port: 8080
      targetPort: 8080
  selector:
      app: demo
  type: ClusterIP
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name:  demo-deployment
  namespace: default
  labels:
    app:  demo
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: demo
    spec:
      terminationGracePeriodSeconds: 35
      containers:
      - name: task
        image: registry.aegonthtf.com/aegonthtf-research/tra-app:23
        imagePullPolicy: Always
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 60
          timeoutSeconds: 6
          successThreshold: 1
          failureThreshold: 5
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 60
          timeoutSeconds: 6
          successThreshold: 1
          failureThreshold: 5
        ports:
          - name: http
            containerPort: 8080
        volumeMounts:
              -
                name: "persistent-storage"
                mountPath: "/appdata"
      volumes:
          -
            name: "persistent-storage"
            persistentVolumeClaim:
            claimName: demo-nas-pvc
```

创建Ingress (什么是Ingress?)

在我们Ocean容器平台架构中，域名及负载均衡反向代理服务器组如果是小区大门的话，那么Ingress就是楼道门。 是k8s集群对外暴露服务的机制，其本质为一个根据定义自动配置和调整的nginx。 有多种实现，但我们目前使用的就是nginx的实现，所以可以这么简单的理解。

容器平台对外暴露服务的路径 client -> nginx -> kubernetes node & ingress -> kubernetes app service -> kubernetes app pod

如下为创建Demo应用的ingress的yaml脚本，执行后，即可创建相应的ingress配置。

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: demo.aegonthtf.com
spec:
  rules:
  - host: demo.aegonthtf.com
    http:
      paths:
      - path: /
        backend:
          serviceName: demo-svc
          servicePort: 8080
```

> #### 服务验证

1. 配置服务域名DNS

在实际工作中应在DNS中增加相关域名配置，但研究阶段也可以在hosts中增加解析。 如上Demo,我在hosts中新增了如下解析。

```
10.72.240.76   demo.aegonthtf.com
```

其中10.72.240.76为我们内部的通用域名、负载均衡、反向代理服务器， 其配置了泛域名转发。 故将符合规定的域名转向它，它可自动将访问转向k8s集群。

1. 访问该域名

![access result ](https://mmbiz.qlogo.cn/mmbiz_png/5Ofd65QfQBDrn9XqAib8CAY8S3Xty6ibDlIfa7Rc258nt8y86PaLBqOlzytDkAEBV73nMDcMMKkgAJZn2ge0j8aA/0?wx_fmt=png)
