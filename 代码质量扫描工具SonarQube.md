
# sonarqube 简介
> 代码扫描工具，可嵌入至基于jenkins pipeline及maven的持续集成部署流程。 检查和提升开发代码质量。

# sonarqube 服务器安装

基于K8S安装sonarqube服务器，数据库也适用在k8s平台上专门为sonarqube部署的postgres数据库。
详细步骤和脚本如下

## 准备镜像
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
## Ceph创建Image, 含sonarqube及postgres db所需持久存储

````bash
rbd create rbd/dev-sonarqube-conf           --size 5G --image-format 2 --image-feature layering
rbd create rbd/dev-sonarqube-data           --size 5G --image-format 2 --image-feature layering
rbd create rbd/dev-sonarqube-extensions     --size 5G --image-format 2 --image-feature layering
rbd create rbd/dev-sonarqube-logs           --size 5G --image-format 2 --image-feature layering
rbd create rbd/dev-sonarqube-bundled-plugins --size 5G --image-format 2 --image-feature layering
rbd create rbd/dev-sonarqube-postgres-storage --size 5G --image-format 2 --image-feature layering
````


## k8s 上创建PV &  PVC

````YAML

````


## k8s上创建postgres db

````YAML

````

> 注意，创建时首次启动会遇到pod无法启动的问题，是因为挂在的Ceph块设备的目录中非空，有一个lost+found的目录，删除后即可正常启动。 删除方法： 可查看postgres pod维护那台机器，登录那台宿主机执行mount命令，查找出mount的目录，登录后，rm -rf lost+found目录，然后数据库即可启动

## k8s上创建 SonarQube  deployment & svc


## k8s 上创建相应的ingress

## 登陆sonarqube，安装相关插件

admin/admin 登录Sonar界面，安装相关的插件，重启

> 与Jenkins的整合，可在webhook中增加所有目标Jenkins的webhooks路径，这样sonarqube即可在扫描计算完毕后，将结果通知到这些Jenkins, 以决定Pipeline是否可以继续。

路径为： http://jenkins-instance/sonarqube-webhooks/


* 参考资料：

1. https://jenkins.io/blog/2017/04/18/continuousdelivery-devops-sonarqube/

2. https://github.com/SonarSource/docker-sonarqube/blob/master/recipes.md?spm=5176.1972344.1.14.kcECvj&file=recipes.md

3. http://www.jumpbeandev.com/2017/01/03/sonarqube/

4. https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+Jenkins

5. https://github.com/felipebz/sonar-plsql






210227a210de6d70bf8fb1b5968ae080a11fa6c0




# Sonar Qube Eclipse使用：



## 使用SonarQube Scanner分析项目代码
http://www.jianshu.com/p/778fd35fd494

SonarQube Scanner，作为代码扫描的工具，通过它，将项目的代码读取并发送至SonarQube服务器中，才能让SonarQube进行代码分析。

可以认为SonarQube Scanner就是SonarQube的客户端。
SonarQube Scanner很方便和不同类型的构建工具进行整合
与Maven项目整合

Maven仓库中就有SonarQube Scanner工具的插件，只要在Setting.conf文件中添加如下配置
````xml
<settings>
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
                  http://myserver:9000
                </sonar.host.url>
            </properties>
        </profile>
     </profiles>
</settings>
````
配置完成后，在项目中，执行mvn sonar:sonar，SonarQube Scanner会自动扫描，根据pom.xml文件得出项目相关信息，不需要自定义sonar-project.properties。扫描完成后就会上传只Sonarqube服务器中。稍后，登陆服务器中就可以看到分析结果了。

## 使用Sonarqube Scanner CLI进行代码扫描

https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner

SonarLint (eclipse plugin)

https://www.sonarlint.org/eclipse/
http://www.jianshu.com/p/778fd35fd494

````java
public class MyClass extends Parent implements Interface {
  public static void main(String[] args) {

  }

}

````
