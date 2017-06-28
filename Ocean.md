# <font color="red">同方全球容器云-Ocean</font>

> 基于Private Cloud,利用Docker及Google Kubernetes提供PAAS平台服务

> 提供DevOPs工具集，包括Gitlab, Jenkins, Sonarqube, Subversion等，实现全新的部署流程，任务流转以及服务全面升级

> 提供Redis,Rabbitmq, Mysql Galara Cluster，Logcenter(ELK)，Prometheus（Monitor)等系统服务

> 支持基于Spring Cloud及Dubbo等分布式框架进行分布式应用开发，基于容器的部署平台不仅从技术上，更从管理成本，资源成本方面为微服务架构应用的开发提供了可行性。

> 支持Spring Boot应用开发。 PAAS平台将提供配置中心， 服务注册／发现中心等基础服务，提供基于Ceph的对象存储服务，使得应用与存储，应用的配置充分解耦，实现应用能在不同环境间轻松迁移。

> 针对Java项目，支持持续集成和部署（定时或提交合并主线时可自动触发部署），支持敏捷开发。 提供质量检查服务，协助提高程序质量。

> 提供多副本的分布式软件定义存储，为应用及系统提供对象存储，块存储以及部分文件存储服务。

> 实现公有云／私有云应用部署上的最大程度相同，实现应用的在私有云／公有云间自由迁移

> 提供资源使用的高度灵活，高密使用，同样资源提供更高的服务能力

> 降低系统管理人员的工作量，将宝贵时间投入到创新或其他高价值工作中


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

  ```bash
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

```docker
# Example instructions from https://docs.docker.com/reference/builder/

FROM ubuntu:14.04

MAINTAINER example@example.com

ENV foo /bar
WORKDIR ${foo}   # WORKDIR /bar
ADD . $foo       # ADD . /bar
COPY \$foo /quux # COPY $foo /quux
ARG   VAR=FOO

RUN apt-get update && apt-get install -y software-properties-common\
    zsh curl wget git htop\
    unzip vim telnet
RUN ["/bin/bash", "-c", "echo hello ${USER}"]

CMD ["executable","param1","param2"]
CMD command param1 param2

EXPOSE 1337

ENV myName="John Doe" myDog=Rex\ The\ Dog \
    myCat=fluffy

ENV myName John Doe
ENV myDog Rex The Dog
ENV myCat fluffy

ADD hom* /mydir/        # adds all files starting with "hom"
ADD hom?.txt /mydir/    # ? is replaced with any single character

COPY hom* /mydir/        # adds all files starting with "hom"
COPY hom?.txt /mydir/    # ? is replaced with any single character
COPY --from=foo / .

ENTRYPOINT ["executable", "param1", "param2"]
ENTRYPOINT command param1 param2

VOLUME ["/data"]

USER daemon

LABEL com.example.label-with-value="foo"
LABEL version="1.0"
LABEL description="This text illustrates \
that label-values can span multiple lines."

WORKDIR /path/to/workdir

ONBUILD ADD . /app/src

STOPSIGNAL SIGKILL

HEALTHCHECK --retries=3 cat /health

SHELL ["/bin/bash", "-c"]
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
