![gitlab](https://mmbiz.qlogo.cn/mmbiz_png/5Ofd65QfQBCpGXCAjcmo94cM95YRZcic7leU4tq60ibfvcGnm0MT20I580fn8aCJtFm4M415eyCssNrIYCPWGckA/0?wx_fmt=png)

# <font color="red"> GitLab Guideline </font>

## Introduction

> **Git** is a free and open source distributed version control system designed to handle everything from small to very large projects with speed and efficiency. Git is easy to learn and has a tiny footprint with lightning fast performance. It outclasses SCM tools like Subversion, CVS, Perforce, and ClearCase with features like cheap local branching, convenient staging areas, and multiple workflows.

> **GitLab** is a web-based Git repository manager with wiki and issue tracking features, using an open source license, developed by GitLab Inc.

如SVN一样，GitLab是一个Git源码管理平台，Git源码管理平台在无论是在开源社区还是在各类组织中都着非常广泛的应用。GitLab不仅是一个源码管理平台，它还有诸如CI/CD持续部署，issue管理等跨越软件生命周期的各类工具和服务。

另外，在容器、DevOPs工具等的结合的实践中，Git比Subversion展现出更多的优势。

## Installation

虽然从Gitlab的官网文档中可以看到为了帮助使用者，GitLab的维护者已经努力的降低复杂性，并且为各类平台都准备了精简的安装步骤。 但毋庸置疑，容器版本是封装最为彻底，安装复杂性最低的一个方式。 所以，我们选择安装容器版本，这也将大大减少未来的维护工作量，无论是故障恢复还是版本升级。

Gitlab分为Community Edition, Enterprise Edition.

> **GitLab Community Edition (CE)** is an opensource product, self-hosted, free to use. All GitLab products contain the features available in GitLab CE. Premium features are available in GitLab Enterprise Edition (EE).

> **GitLab Enterprise Edition (EE)** is an opencore product, self-hosted, available under distinct subscriptions. GitLab EE contains all the features available in GitLab Community Edition (CE), plus premium features available in each version: Enterprise Edition Starter (EES) and Enterprise Edition Premium (EEP). Everything available in EES is also available in EEP.

> **Omnibus** is a way to package different services and tools required to run GitLab, so that most users can install it without laborious configuration.

Gitlab的容器镜像是使用Omnibus生成的，包含了构建Gitlab平台的各个组建和服务，最大程度的封装复杂性，使得安装尽可能的简便。

我们在k8s平台上使用Gitlab官方镜像安装Gitlab平台。 持久存储将使用Ceph对象存储。

### 安装过程

1. 在ceph存储上为giblab容器创建持久存储。

  ```
    rbd create rbd/gitlab-postgre-storage --size 5G --image-format 2 --image-feature layering
    rbd create rbd/gitlab-postgre-storage --size 5G --image-format 2 --image-feature layering
    rbd create rbd/gitlab-postgre-storage --size 5G --image-format 2 --image-feature layering
    rbd create rbd/gitlab-postgre-storage --size 5G --image-format 2 --image-feature layering
    rbd create rbd/gitlab-postgre-storage --size 5G --image-format 2 --image-feature layering
  ```

2. 在k8s中为gitlab创建pv & pvc

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: gitlab-data-pv   
spec:
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  rbd:
    monitors:
      - 10.72.243.113:6789,10.72.243.114:6789,10.72.243.115:6789
    pool: rbd
    image: gitlab-storage
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
  name: gitlab-data-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: gitlab-config-pv
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  rbd:
    monitors:
      - 10.72.243.113:6789,10.72.243.114:6789,10.72.243.115:6789
    pool: rbd
    image: gitlab-config-storage
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
  name: gitlab-config-claim

spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: gitlab-log-pv     
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  rbd:
    monitors:
      - 10.72.243.113:6789,10.72.243.114:6789,10.72.243.115:6789
    pool: rbd
    image: gitlab-log-storage
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
  name: gitlab-log-claim     
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
---  
apiVersion: v1
kind: PersistentVolume
metadata:
  name: gitlab-postgres-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  rbd:
    monitors:
      - 10.72.243.113:6789,10.72.243.114:6789,10.72.243.115:6789
    pool: rbd
    image: gitlab-postgre-storage
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
  name: gitlab-postgres-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
---      
apiVersion: v1
kind: PersistentVolume
metadata:
  name: gitlab-redis-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  rbd:
    monitors:
      - 10.72.243.113:6789,10.72.243.114:6789,10.72.243.115:6789
    pool: rbd
    image: gitlab-redis-storage
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
  name: gitlab-redis-claim

spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi      
```

3. 在k8s中为gitlab创建专用Redis服务（未构建集群）

```
apiVersion: v1
kind: Service
metadata:
  name: gitlab-redis
  labels:
    name: gitlab-redis
spec:
  selector:
    name: gitlab-redis
  ports:
    - name: redis
      port: 6379
      targetPort: redis
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: gitlab-redis
spec:
  replicas: 1
  template:
    metadata:
      labels:
        name: gitlab-redis
    spec:
      containers:
      - name: redis
        image: registry.aegonthtf.com/library/redis
        ports:
        - name: redis
          containerPort: 6379
        volumeMounts:
        - mountPath: /var/lib/redis
          name: data
        livenessProbe:
          exec:
            command:
            - redis-cli
            - ping
          initialDelaySeconds: 30
          timeoutSeconds: 5
        readinessProbe:
          exec:
            command:
            - redis-cli
            - ping
          initialDelaySeconds: 5
          timeoutSeconds: 1
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: gitlab-redis-claim          
```
4. 在k8s中为gitlab创建postgresql数据库服务

**重要提示**  因postgresql在初始化时，要求其使用的存储路径全空，而Ceph如上创建提供的存储中包含lost+found文件夹，不作处理则容器启动会失败。所以，在创建和启动postgresql DB Container之前，首先先登陆Ceph服务器，通过如下命令，将lost+found文件夹删除。

```
rbm map  gitlab-postgre-storage
mount  dev/..   /mnt
rm -rf lost+found
umount /mnt

```

通过如下脚本创建和启动PostgreSQL DB.

```
apiVersion: v1
kind: Service
metadata:
  name: gitlab-postgresql
  labels:
    name: gitlab-postgresql
spec:
  ports:
    - name: postgres
      port: 5432
      targetPort: postgres
  selector:
    name: gitlab-postgresql
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: gitlab-postgresql
spec:
  replicas: 1
  template:
    metadata:
      labels:
        name: gitlab-postgresql
    spec:
      containers:
      - name: postgresql
        image: registry.aegonthtf.com/library/postgres
        imagePullPolicy: Always
        env:
        - name: POSTGRES_USER
          value: gitlab
        - name: POSTGRES_PASSWORD
          value: +BP52QIxpT/flVCMpL3KXA==
        - name: POSTGRES_DB
          value: gitlab_production
        - name: DB_EXTENSION
          value: pg_trgm
        ports:
        - name: postgres
          containerPort: 5432
        volumeMounts:
        - mountPath: /var/lib/postgresql/data         
          name: data
        livenessProbe:
          exec:
            command:
            - pg_isready
            - -h
            - localhost
            - -U
            - postgres
          initialDelaySeconds: 30
          timeoutSeconds: 5
        readinessProbe:
          exec:
            command:
            - pg_isready
            - -h
            - localhost
            - -U
            - postgres
          initialDelaySeconds: 5
          timeoutSeconds: 1
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: gitlab-postgres-claim
```
5. 在k8s中安装gitlab平台

通过如下脚本安装gitlab.

```
apiVersion: v1
kind: Service
metadata:
  name: gitlab  
  labels:
    name: gitlab
spec:
  type: LoadBalancer
  selector:
    name: gitlab
  ports:
    - name: http
      port: 80
      targetPort: http
    - name: ssh
      port: 1022
      targetPort: ssh
---      
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: gitlab  
spec:
  replicas: 1
  template:
    metadata:
      labels:
        name: gitlab
        app: gitlab
    spec:
      containers:
      - name: gitlab
        image: registry.aegonthtf.com/library/gitlab/gitlab-ce
        imagePullPolicy: Always
        env:
        - name: GITLAB_OMNIBUS_CONFIG
          value: |
            external_url "http://gitlab.aegonthtf.com"
            postgresql['enable']=false
            gitlab_rails['db_host'] = 'gitlab-postgresql'
            gitlab_rails['db_password']='+BP52QIxpT/flVCMpL3KXA=='
            gitlab_rails['db_username']='gitlab'
            gitlab_rails['db_database']='gitlab_production'
            redis['enable'] = false
            gitlab_rails['redis_host']='gitlab-redis'
            manage_accounts['enable'] = true
            manage_storage_directories['manage_etc'] = false
            gitlab_shell['auth_file'] = '/var/opt/gitlab/ssh/authorized_keys'
            git_data_dir '/var/opt/gitlab/git-data'
            gitlab_rails['shared_path'] = '/var/opt/gitlab/shared'
            gitlab_rails['uploads_directory'] = '/var/opt/gitlab/uploads'
            gitlab_ci['builds_directory'] = '/var/opt/gitlab/builds'
        ports:
        - name: http
          containerPort: 80
        - name: ssh
          containerPort: 22        
        volumeMounts:
        - name: gitlab-data-storage
          mountPath: /var/opt/gitlab
        - name: gitlab-log-storage
          mountPath: /var/log/gitlab
        - name: gitlab-config-storage
          mountPath: /etc/gitlab
        livenessProbe:
          httpGet:
            path: /help
            port: 80
          initialDelaySeconds: 180
          timeoutSeconds: 15
        readinessProbe:
          httpGet:
            path: /help
            port: 80
          initialDelaySeconds: 15
          timeoutSeconds: 1
      volumes:
      - name: gitlab-data-storage
        persistentVolumeClaim:
          claimName: gitlab-data-claim
      - name: gitlab-config-storage
        persistentVolumeClaim:
          claimName: gitlab-config-claim
      - name: gitlab-log-storage
        persistentVolumeClaim:
          claimName: gitlab-log-claim        

```
5. 在k8s中通过ingress暴露服务，ingress中的host设置为gitlab.example.org

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: gitlab-ingress
#  annotations:
#    nginx.org/rewrites: "serviceName=gitlab rewrite=/;serviceName=gitlab rewrite=/gitlab/users/"
spec:
  rules:
  - host: gitlab.aegonthtf.com
    http:
      paths:
      - path: /gitlab
        backend:
          serviceName: gitlab
          servicePort: 80
```

6. 设置gitlab.example.org域名指向nginx服务器（域名服务器，keeplived ha) , 在nginx设置了泛域名反向代理（待验证），反向代理至k8s各node的80端口。

```
server {
    listen       80;
    server_name  gitlab.aegonthtf.com logcenter-dev.aegonthtf.com jenkins-edge.aegonthtf.com zipkin-dev.aegonthtf.com earthcore.aegonthtf.com sonarqube-dev.aegonthtf.com eureka-dev.aegonthtf.com ;  

    #charset koi8-r;
    access_log  /var/log/nginx/gitlab.log  main;

    location / {
       #index  index.html index.htm;
       proxy_set_header Host $http_host;
       proxy_pass http://k8s-standalone-backup;
       client_max_body_size 200m;
       client_body_buffer_size 128k;
    }
}
```

7. 相关界面

* gitlab在kubernetes上部署的组件

![部署界面](https://mmbiz.qlogo.cn/mmbiz_jpg/5Ofd65QfQBBCT79ibpa1YtCcjtv6aAnrBp4nrNLen0E0RFNuvicfnnmOD2eickfbLYTwD1jKGibA6QPxEvBwPAyV5A/0?wx_fmt=jpeg)

* 登录界面
![gitlab 登录界面](https://mmbiz.qlogo.cn/mmbiz_jpg/5Ofd65QfQBBCT79ibpa1YtCcjtv6aAnrBBLNOgSAic1rH0WVOzzsJUmibvKkWc0miaGqcMrw6fiaBIXuoavktmhnYSg/0?wx_fmt=jpeg)

* gitlab主界面

![gitlab主界面](https://mmbiz.qlogo.cn/mmbiz_jpg/5Ofd65QfQBBCT79ibpa1YtCcjtv6aAnrBcAksTF2xicawX5lFmwI55h6AAxZuQKIcXdt2X03WX01y8dLrOiceFkPA/0?wx_fmt=jpeg)

* gitlab创建Group界面

![创建Group](https://mmbiz.qlogo.cn/mmbiz_jpg/5Ofd65QfQBBCT79ibpa1YtCcjtv6aAnrBxl5KupQYQE9xOx4YDhRYEKqv1Jvsc7zAT3XA37bSGgTkJERA8pbugA/0?wx_fmt=jpeg)

* 授予其他开发者权限

![授予其他开发者权限](https://mmbiz.qlogo.cn/mmbiz_jpg/5Ofd65QfQBBCT79ibpa1YtCcjtv6aAnrB8JppJYv0uNhuULhs0RBricibxvOTYfPAYOw6Oo1ZnQcCqZp14hB6AvyA/0?wx_fmt=jpeg)

* 创建项目

![创建项目](https://mmbiz.qlogo.cn/mmbiz_jpg/5Ofd65QfQBBCT79ibpa1YtCcjtv6aAnrBm8Anl2b0wMmYG0RKLNFspiciaRaL7nYrLA68qAB9842n6W5PrcqVHiceQ/0?wx_fmt=jpeg)

* 配置项目

配置项目，使得相关的gitlab事件能够传送到jenkins. 比如当有变更提交至项目Master分支时，触发Jenkins进行编译和部署。

![配置项目，集成](https://mmbiz.qlogo.cn/mmbiz_jpg/5Ofd65QfQBBCT79ibpa1YtCcjtv6aAnrB3vhUvC3bGys2uAXI77m8wYEsACkbibAS7apsed9Y8b5O2SLicPTCQlEw/0?wx_fmt=jpeg)



## Usage Guideline

### 申请账号
### 创建Group
### 创建项目
###

## Tricks

### Account Apply And Authority Management

Gitlab 账号可自我管理，开发人员自行申请，由项目或系统负责人授予或回收权限。

1. 申请账号
2. 管理权限

# Reference

[Gitlab CE Document][821c0ecb]

[Gitlab Docker Image And Installation][83d48fe9]

[821c0ecb]: https://docs.gitlab.com/ce/README.html "Gitlab CE Document"
[83d48fe9]: https://docs.gitlab.com/omnibus/docker/README.html#gitlab-docker-images "Gitlab Docker Image And Installation"
