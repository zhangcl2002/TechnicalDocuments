# <font color="red">River</font>

持续集成部署平台

> 持续集成 持续部署

> 敏捷开发的基础

> 软件生命周期中的自动化

> 提升运维人员工作效率，提升价值的利器

## 平台组件

- Subversion / Gitlab

- Jenkins

- Sonarqube

- Kubernetes
- Docker

## 平台架构及拓扑

## 各组件安装

### Jenkins安装

Jenkins 将以容器安装的方式部署在Kubernetes平台上，以最简化安装过程以及后续维护。后台存储选用Ceph分布式存储，以利用其多副本等高可靠以及灵活扩展的特性。

Jenkins镜像我们采用定制化镜像，该镜像包含了我们选用的Oracle JDK, Mysql Client, Maven等组件。

安装过程如下：

1. 持久化存储创建 登陆Ceph服务器，然后执行下述指令

  ```shell
  rbd create rbd/jenkins-home --size 20G --image-format 2 --image-feature layering
  ```

  **_注：_** 使用Ceph作为持久化存储，当分配的存储不足时，可使用Ceph命令对其进行扩容，可参考 [ceph rdb image resize](http://www.mamicode.com/info-detail-1480817.html)

2. Kubernetes PV & PVC 创建

  在kubernetes Dashboard以图形化界面方式使用下述文件创建或者上传文件到Kubernetes Master后执行

```yaml
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

3\. 在kubernetes平台上创建Jenkins

**_注意：_** **_Jenkins Base Image Dockerfile_**

(该文件尚待修改，目前使用当前使用的定制版本)

```docker

FROM docker.io/library/Jenkins

ADD  ORALCE JAVA (选那个JDK需要考虑)
ADD  MYSQL (为未来执行Mysql Script)
ADD  Maven
```

Jenins.yaml

```yaml
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

通过Scope登入或者是从Jenkins 所在容器的宿主机上使用 Docker exec -it .. bash 命令登入，或者是从kubernetes Master上使用kubectl exec -it pod-name bash 登入Jenkins容器，从下图所示的文件中获得安装的密码。

获得密码后，在首次登陆Jenkins时按照提示添入。

![Jenkins SVC Image](https://mmbiz.qlogo.cn/mmbiz_png/5Ofd65QfQBBCT79ibpa1YtCcjtv6aAnrBDcxoRKuTIaiazAAfH2V4QrYxw01IMUYI6jaiaNVIX4eFuhodZRJ0MwSw/0?wx_fmt=png)

![Fill the initial password](https://mmbiz.qlogo.cn/mmbiz_png/5Ofd65QfQBBCT79ibpa1YtCcjtv6aAnrBcUQh98I1xL4QNHM7UyKnqJWr5fCeCicoHJxDr7VOpv7y2CUVSSiad2Gg/0?wx_fmt=png)

为了使得Jenkins能够联网安装Plugin,需要在设置代理（除非服务器能直接上网）（此处docker是否可以联网是否有关系未做测试，推测应该两者无关系）

![setup the proxy](https://mmbiz.qlogo.cn/mmbiz_png/5Ofd65QfQBBCT79ibpa1YtCcjtv6aAnrBlf5rGb5jtVggLrbTS9warQJZJr7xS4QvqeYgz1LQ2ZeGJJ3piaicc0FA/0?wx_fmt=png)

![Install the recommend plugin](https://mmbiz.qlogo.cn/mmbiz_png/5Ofd65QfQBBCT79ibpa1YtCcjtv6aAnrBCJNbh3ChWPLeFb47icELyUePbJ6a5QHa5ojkstRzuGe7fUN60icQxN1w/0?wx_fmt=png)

**_目前未总结在River持续集成平台中安装了那些插件，这些截图中的插件供参考_**

![Install the recommend plugin](https://mmbiz.qlogo.cn/mmbiz_png/5Ofd65QfQBBCT79ibpa1YtCcjtv6aAnrBYribFNZKsr3zqz84hErTczAvhyddLkQvZ5VKO3k4L1ibHfxCj0uSq0Hw/0?wx_fmt=png)

![Install the recommend plugin](https://mmbiz.qlogo.cn/mmbiz_png/5Ofd65QfQBBCT79ibpa1YtCcjtv6aAnrBZXAGvlNRzctic6Tg2ibG6FZuv8lakUzkJGghAvNG1Z9Qwg2qcicY8CApg/0?wx_fmt=png)

![Install the recommend plugin](https://mmbiz.qlogo.cn/mmbiz_png/5Ofd65QfQBBCT79ibpa1YtCcjtv6aAnrBDz37bvJE3q6x85JZicCtGa3N7LgdLibqN8DJlOvWiaWWYlmvtwVhrPWJw/0?wx_fmt=png)

### Jenkins配置

- Unlock

- Proxy Setting
- Install Plugins
- Create Admin Accounts
- General Setting

### Jenkins Pipeline

### Use Case
