
# SonarQube 代码质量检查平台的安装部署和使用指南

代码扫描工具，可用于开发过程中检查和控制代码质量，也可嵌入至基于jenkins pipeline及maven的持续集成部署流程，在上线之前检查和控制代码质量。亦可用于QA检查控制代码质量，编制报告之用。

本笔记包含两部分内容：

1. SonarQube服务器安装和配置步骤
2. 预期使用场景中的使用指南

## sonarqube 服务器安装指南

基于K8S安装专用于SonarQube的postgre DB服务器以及SonarQube应用服务器。
详细步骤和脚本如下

### 准备镜像
````bash
[root@vm-shaldk99 ~]# docker pull sonarqube:6.6
[root@vm-shaldk99 ~]# docker run -d --name sonarqube -p 9000:9000 -p 9092:9092 sonarqube
[root@vm-shaldk99 ~]# docker cp 1e44b92968f22e06b74bc6697cd8aa1cc896fb227481071afde06f895efee640:/opt/sonarqube/conf/sonar.properties  sonar.properties
[root@vm-shaldk99 ~]# vi sonar.properties

````
修改sonar.properties,增加代理设置。（后面在下载插件的时候会需要上网）
````properties
# UPDATE CENTER
# Update Center requires an internet connection to request https://update.sonarsource.org
# It is enabled by default.
sonar.updatecenter.activate=true
# HTTP proxy (default none)
http.proxyHost=10.72.1.253
http.proxyPort=8080
# HTTPS proxy (defaults are values of http.proxyHost and http.proxyPort)
https.proxyHost=10.72.1.253
https.proxyPort=8080
# NT domain name if NTLM proxy is used
http.auth.ntlm.domain=aegonthtf
# SOCKS proxy (default none)
#socksProxyHost=
#socksProxyPort=
# Proxy authentication (used for HTTP, HTTPS and SOCKS proxies)
http.proxyUser=user
http.proxyPassword=password
````
````
[root@vm-shaldk99 ~]# docker cp sonar.properties 1e44b92968f22e06b74bc6697cd8aa1cc896fb227481071afde06f895efee640:/opt/sonarqube/conf/sonar.properties
[root@vm-shaldk99 ~]# docker commit 1e44b92968f22e06b74bc6697cd8aa1cc896fb227481071afde06f895efee640 registry.aegonthtf.com/library/sonarqube:6.6

````
### 准备持久存储
* Ceph创建Image, 含sonarqube及postgres db所需持久存储

````bash
rbd create rbd/dev-sonarqube-conf           --size 5G --image-format 2 --image-feature layering
rbd create rbd/dev-sonarqube-data           --size 5G --image-format 2 --image-feature layering
rbd create rbd/dev-sonarqube-extensions     --size 5G --image-format 2 --image-feature layering
rbd create rbd/dev-sonarqube-logs           --size 5G --image-format 2 --image-feature layering
rbd create rbd/dev-sonarqube-bundled-plugins --size 5G --image-format 2 --image-feature layering
rbd create rbd/dev-sonarqube-postgres-storage --size 5G --image-format 2 --image-feature layering
````

* k8s 上创建PV &  PVC

````YAML
apiVersion: v1
kind: PersistentVolume
metadata:
  name: dev-sonarqube-conf-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  rbd:
    monitors:
      - 10.72.243.201:6789,10.72.243.203:6789,10.72.243.205:6789
    pool: rbd
    image: dev-sonarqube-conf
    user: admin
    secretRef:
      name: ceph-prod-secret
    fsType: ext4
    readOnly: false
  persistentVolumeReclaimPolicy: Recycle
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: dev-sonarqube-conf-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: dev-sonarqube-data-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  rbd:
    monitors:
      - 10.72.243.201:6789,10.72.243.203:6789,10.72.243.205:6789
    pool: rbd
    image: dev-sonarqube-data
    user: admin
    secretRef:
      name: ceph-prod-secret
    fsType: ext4
    readOnly: false
  persistentVolumeReclaimPolicy: Recycle
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: dev-sonarqube-data-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: dev-sonarqube-extensions-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  rbd:
    monitors:
      - 10.72.243.201:6789,10.72.243.203:6789,10.72.243.205:6789
    pool: rbd
    image: dev-sonarqube-extensions
    user: admin
    secretRef:
      name: ceph-prod-secret
    fsType: ext4
    readOnly: false
  persistentVolumeReclaimPolicy: Recycle
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: dev-sonarqube-extensions-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: dev-sonarqube-logs-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  rbd:
    monitors:
      - 10.72.243.201:6789,10.72.243.203:6789,10.72.243.205:6789
    pool: rbd
    image: dev-sonarqube-logs
    user: admin
    secretRef:
      name: ceph-prod-secret
    fsType: ext4
    readOnly: false
  persistentVolumeReclaimPolicy: Recycle
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: dev-sonarqube-logs-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: dev-sonarqube-bundled-plugins-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  rbd:
    monitors:
      - 10.72.243.201:6789,10.72.243.203:6789,10.72.243.205:6789
    pool: rbd
    image: dev-sonarqube-bundled-plugins
    user: admin
    secretRef:
      name: ceph-prod-secret
    fsType: ext4
    readOnly: false
  persistentVolumeReclaimPolicy: Recycle
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: dev-sonarqube-bundled-plugins-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
````


### 构建SonarQube所用的数据库（此处选用postgres)

* k8s上创建postgres db

````YAML
apiVersion: v1
kind: PersistentVolume
metadata:
  name: dev-sonarqube-postgres-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  rbd:
    monitors:
      - 10.72.243.201:6789,10.72.243.203:6789,10.72.243.205:6789
    pool: rbd
    image: dev-sonarqube-postgres-storage
    user: admin
    secretRef:
      name: ceph-prod-secret
    fsType: ext4
    readOnly: false
  persistentVolumeReclaimPolicy: Recycle
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: dev-sonarqube-postgres-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
---
apiVersion: v1
kind: Service
metadata:
  name: dev-sonarqube-postgres
  labels:
    name: dev-sonarqube-postgres
spec:
  ports:
    - name: postgres
      port: 5432
      targetPort: postgres
  selector:
    name: dev-sonarqube-postgres
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: dev-sonarqube-postgres
spec:
  replicas: 1
  template:
    metadata:
      labels:
        name: dev-sonarqube-postgres
    spec:
      containers:
      - name: dev-sonarqube-postgres
        image: registry.aegonthtf.com/library/postgres:10
        imagePullPolicy: Always
        env:
        - name: POSTGRES_USER
          value: sonarqube
        - name: POSTGRES_PASSWORD
          value: sonarqube
        - name: POSTGRES_DB
          value: sonarqube_db
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
          claimName: dev-sonarqube-postgres-claim

````

> 注意，创建时首次启动会遇到pod无法启动的问题，是因为挂在的Ceph块设备的目录中非空，有一个lost+found的目录，删除后即可正常启动。 删除方法： 可查看postgres pod维护那台机器，登录那台宿主机执行mount命令，查找出mount的目录，登录后，rm -rf lost+found目录，然后数据库即可启动

### 在容器云上创建SonarQube服务器

* k8s上创建 SonarQube  deployment & svc
````yaml
apiVersion: v1
kind: Service
metadata:
  name: dev-sonarqube-svc
  labels:
    name: dev-sonarqube
spec:
  ports:
    - name: dev-sonarqube
      port: 9000
      targetPort: 9000
  selector:
    name: dev-sonarqube
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: dev-sonarqube-deployment
spec:
  replicas: 1
  template:
    metadata:
      labels:
        name: dev-sonarqube
    spec:
      containers:
      - name: dev-sonarqube
        image: registry.aegonthtf.com/library/sonarqube:6.6
        imagePullPolicy: Always
        env:
        - name: SONARQUBE_JDBC_USERNAME
          value: sonarqube
        - name: SONARQUBE_JDBC_PASSWORD
          value: sonarqube
        - name: SONARQUBE_JDBC_URL
          value: 'jdbc:postgresql://dev-sonarqube-postgres:5432/sonarqube_db'
        ports:
        - name: sonarqube
          containerPort: 9000
        volumeMounts:
        - mountPath: /opt/sonarqube/conf
          name: sonarqube-conf
        - mountPath: /opt/sonarqube/data
          name: sonarqube-data
        - mountPath: /opt/sonarqube/extensions
          name: sonarqube-extensionsh
        - mountPath: /opt/sonarqube/logs
          name: sonarqube-logs
        - mountPath: /opt/sonarqube/lib/bundled-plugins
          name: sonarqube-bundled-plugins
      volumes:
      - name: sonarqube-conf
        persistentVolumeClaim:
          claimName: dev-sonarqube-conf-claim
      - name: sonarqube-data
        persistentVolumeClaim:
          claimName: dev-sonarqube-data-claim
      - name: sonarqube-extensionsh
        persistentVolumeClaim:
          claimName: dev-sonarqube-extensions-claim
      - name: sonarqube-logs
        persistentVolumeClaim:
          claimName: dev-sonarqube-logs-claim
      - name: sonarqube-bundled-plugins
        persistentVolumeClaim:
          claimName: dev-sonarqube-bundled-plugins-claim
````

   * k8s 上创建相应的ingress

### SonarQube服务器安装后设置

 * 登陆sonarqube，安装相关插件

 * admin/admin 登录Sonar界面，安装相关的插件，重启

### 如何将Sonarqube增加进持续集成部署流程

与Jenkins的整合，我们选用Pipeline的方式。 在SonarQube服务器上的webhook中增加所有目标Jenkins的webhooks路径，这样sonarqube可在扫描计算完毕后，将结果通知到这些Jenkins, 以决定Pipeline是否可以继续。

路径为： http://jenkins-instance/sonarqube-webhooks/

此处是未来控制代码质量的关键节点。 （如果代码质量检查是真的有效、有用）



## Sonar Qube 使用指南

### 使用场景
   1. 开发过程中检查分析代码质量，确保产出代码的质量
   2. 部署过程中分析检查代码质量，并增加检查标准和阀值，不符合质量要求的代码不允许部署到UAT环境和生产环境
   3. 调阅代码质量检查的数据，编写QA质量检查报告



### 开发过程中如何使用

    1. 在eclipse中使用Maven分析项目代码质量

http://www.jianshu.com/p/778fd35fd494

SonarQube Scanner，是sonarqube是SonarQube的客户端,是代码扫描的工具，它会将项目的代码进行分析并与sonarqube Server一起完成代码分析，最后将结果发送至SonarQube服务器。

SonarQube Scanner很方便和不同类型的构建工具进行整合，与maven的integrate的方式如下：

Maven仓库中有SonarQube Scanner工具的插件，只需要进行一下设置即可使用。在我们这里需要在Setting.conf文件进行如下配置（该配置指定了本地Maven仓库，并设置完毕sonarqube相关信息）
````xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.1.0http://maven.apache.org/xsd/settings-1.1.0.xsd";; xmlns="http://maven.apache.org/SETTINGS/1.1.0";;
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">;;

  <pluginGroups>
    <pluginGroup>org.sonarsource.scanner.maven</pluginGroup>
  </pluginGroups>

  <profiles>
    <profile>
        <id>sonar</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <!-- Optional URL to server. Default value is http://localhost:9000 -->
            <sonar.host.url>
                 http://sonarqube-dev.aegonthtf.com
            </sonar.host.url>
        </properties>
     </profile>

    <profile>
      <repositories>
        <repository>
          <snapshots>
            <enabled>false</enabled>
          </snapshots>
          <id>central</id>
          <name>libs-release</name>
          <url>http://artifactory.aegonthtf.com/artifactory/libs-release</url>;;
        </repository>
        <repository>
          <snapshots />
          <id>snapshots</id>
          <name>libs-snapshot</name>
          <url>http://artifactory.aegonthtf.com/artifactory/libs-snapshot</url>;;
        </repository>
      </repositories>
      <pluginRepositories>
        <pluginRepository>
          <snapshots>
            <enabled>false</enabled>
          </snapshots>
          <id>central</id>
          <name>plugins-release</name>
          <url>http://artifactory.aegonthtf.com/artifactory/plugins-release</url>;;
        </pluginRepository>
        <pluginRepository>
          <snapshots />
          <id>snapshots</id>
          <name>plugins-snapshot</name>
          <url>http://artifactory.aegonthtf.com/artifactory/plugins-snapshot</url>;;
        </pluginRepository>
      </pluginRepositories>
      <id>artifactory</id>
    </profile>
  </profiles>
  <activeProfiles>
    <activeProfile>artifactory</activeProfile>
  </activeProfiles>
</settings>

````
配置完成后，在项目中，执行mvn sonar:sonar，SonarQube Scanner会自动扫描，根据pom.xml文件得出项目相关信息，不需要自定义sonar-project.properties。扫描完成后就会上传只Sonarqube服务器中。稍后，登陆服务器中就可以看到分析结果了。


一个实际的执行示例：



### SonarLint (eclipse plugin)组件的安装和使用

SonarLint可以在开发工具中实时对代码进行分析，并可以与sonarqube服务器联系。 试用过程发现sonarlint的分析结果并不会上传到sonarqube服务器中，但服务器中的策略等会影响sonarlint在本地的分析表现。
比如： 代码中存在一个已经被探知的bug,如果在服务器上标志已经修复，在下一次Sonarlint在分析时就会忽略这个bug.

https://www.sonarlint.org/eclipse/
http://www.jianshu.com/p/778fd35fd494

### 使用Sonarqube Scanner CLI进行代码扫描(作者未测试这个部分)

https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner


* 参考资料：

1. https://jenkins.io/blog/2017/04/18/continuousdelivery-devops-sonarqube/

2. https://github.com/SonarSource/docker-sonarqube/blob/master/recipes.md?spm=5176.1972344.1.14.kcECvj&file=recipes.md

3. http://www.jumpbeandev.com/2017/01/03/sonarqube/

4. https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+Jenkins

5. https://github.com/felipebz/sonar-plsql
