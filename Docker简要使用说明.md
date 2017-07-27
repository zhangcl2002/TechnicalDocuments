
> ### 本文目标

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

> ### 主要步骤
<br>
> #### 镜像构建
1. 了解基础镜像
   Docker的镜像是分层的，一层一层叠加的。比如，最底层就是scratch, 意思就是什么也没有，然后上面可以是CentOS, 然后上面是Tomcat,然后再上面是将实际的业务应用以及对Tomcat的定制化内容加入，最终形成一个实际可以使用的应用系统的Docker镜像。
2. 编写Dockerfile文件

   Dockerfile是生成Docker镜像的脚本。

   比如CentOS： https://github.com/CentOS/sig-cloud-instance-images/blob/5bbaef9f60ab9e3eeb61acec631c2d91f8714fff/docker/Dockerfile

   比如Tomcat：
   https://github.com/docker-library/tomcat/blob/e45349ec3432c75ca5b7e2ef4cae95276c7dccc7/8.0/jre8/Dockerfile


3. 生成镜像并推送至镜像仓库
将所有相关的文件放在安装有Docker Engine的服务器上某个文件夹下，然后在该文件夹执行下面的命令

```
docker build -t dockername .
```

> #### 持久存储配备
容器可以使用多种持久存储，比如本地硬盘，NAS共享文件存储，Ceph块存储，Ceph文件存储。 此处以Ceph存储举例。
1. 在持久存储上创建需要的存储位置
创建ceph 块存储image

登陆可以操作ceph的机器，创建ceph image (这是简要的使用，实际更完善的使用访问以后再逐步完善)
`rbd create rbd/demo-storage --size 5G --image-format 2 --image-feature layering`
查看
`rbd ls`
`rbd info demo-storage`
删除
`rbd rm demo-storage`

2. 在k8s中创建PV & PVC
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: demo-pv
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
  name: demo-pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

> #### 在k8s上部署服务
1. 通过k8s Dashboard界面部署
2. 通过yaml文件部署服务

NAS存储
```
kind: Service
apiVersion: v1
metadata:
  name: $2-svc
spec:
  ports:
    - name: http
      port: 8080
      targetPort: 8080
  selector:
      app: $2
  type: ClusterIP
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name:  $2-deployment
  namespace: default
  labels:
    app:  $2
spec:
  replicas: $3
  template:
    metadata:
      labels:
        app: $2
    spec:
      terminationGracePeriodSeconds: 35
      containers:
      - name: $2
        image: registry.aegonthtf.com/$1/$2:$9
        imagePullPolicy: Always
        readinessProbe:
          httpGet:
            path: $8
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 60
          timeoutSeconds: 6
          successThreshold: 1
          failureThreshold: 5
        livenessProbe:
          httpGet:
            path: $8
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
                    name: "nas-directory"
                    mountPath: "/appdata"
        volumes:
              -
                name: "nas-directory"
                hostPath:
                  path: "/appdata/"
```
ceph存储
```
```



```ingress
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: $7
spec:
  rules:
  - host: $7
    http:
      paths:
      - path: $8
        backend:
          serviceName: $2-svc
          servicePort: 8080
```
> #### 服务验证

1. 配置服务域名DNS

2. 访问该域名
