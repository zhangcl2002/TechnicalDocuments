
![Ocean](https://mmbiz.qlogo.cn/mmbiz_png/5Ofd65QfQBAK7QQJXOwPcvvOcnhG4hRJvAiaNBPCnDpccfULQqtMR1VLFKX63Aenf1BziaQwQibg6NUYSmUbdibJnw/0?wx_fmt=png)
<br />
# <font color="#003399">同方全球容器云-Ocean</font>
<br />

<font color="#FF6666">
小伙伴们差不多花了一年的时间投入在容器云的建设上，大家有一个一致的目标，就是期望通过我们的努力和智慧，将孵化成熟的新科技引入到工作实践中，触发变化，实现可能，创造价值。
<br>
纲举则目张。本文希望做一个引子，可窥全貌。用心于每一个细节但始终胸有全图，每一场战斗，一城一池的争夺，坚持不放弃，将决定战役的最终胜利。
</font>

<br />
## Ocean Platform 用途
<br />
> *基于Private Cloud,利用Docker及Google Kubernetes构建提供PAAS平台服务*
> *该平台将提供给应用无以伦比的灵活性，高密利用资源，使得灵活扩缩，自我监控，自我愈合，灰度发布等互联网应用能力在传统企业也同样实现*
> *我们认为未来商业外购应用也将越来越多的采用容器的方式发布，完全封装安装复杂性，升级替换等维护动作变成一件简单的事情*

<br />
> *提供DevOPs工具集，包括Gitlab, Jenkins, Sonarqube, Subversion等，实现全新的部署流程，任务流转以及服务全面升级*

<br />
> *提供Redis,Rabbitmq, Mysql Galara Cluster，Logcenter(ELK)，Prometheus（Monitor),Harbor Registry等系统服务*

<br />
> *支持基于Spring Cloud及Dubbo等分布式框架进行分布式应用开发，基于容器的部署平台不仅从技术上，更从管理成本，资源成本方面为微服务架构应用的开发提供了可行性。*

<br />
> *支持Spring Boot应用开发。 PAAS平台将提供配置中心， 服务注册／发现中心等基础服务，提供基于Ceph的对象存储服务，使得应用与存储，应用的配置充分解耦，实现应用能在不同环境间轻松迁移。*

<br />
> *针对Java项目，支持持续集成和部署（定时或提交合并主线时可自动触发部署），支持敏捷开发。 提供质量检查服务，协助提高程序质量。*

<br />
> *提供多副本的分布式软件定义存储，为应用及系统提供对象存储，块存储以及部分文件存储服务。*

<br />
> *实现公有云／私有云应用部署上的最大程度相同，实现应用的在私有云／公有云间自由迁移*

<br />
> *提供资源使用的高度灵活，高密使用，同样资源提供更高的服务能力*

<br />
> *降低系统管理人员的工作量，将宝贵时间投入到创新或其他高价值工作中*

<br />

## Ocean平台的组成

- Docker
- Kubernetes
![kubernetes](https://mmbiz.qlogo.cn/mmbiz_png/5Ofd65QfQBCLAg5aiazhvYYa9OGibBoJzy10uOzoZ2gibDic6iaxgak4MIZFwnnicyBEuodwLp8tiaiaZtHf3bibZ4eHx4w/0?wx_fmt=png)
- Ceph
![ceph dashboard ](https://mmbiz.qlogo.cn/mmbiz_png/5Ofd65QfQBCLAg5aiazhvYYa9OGibBoJzypQia9UnPaRibkL2A0zTyZAs5vvAHUsJicVjSo0MnECCZJ0XRkK7Jz5MwA/0?wx_fmt=png)
- Harbor Registry
![Harbor](https://mmbiz.qlogo.cn/mmbiz_png/5Ofd65QfQBCLAg5aiazhvYYa9OGibBoJzyYo1cD1LJGSJXPEDHdtDiaSoTmsI8vckYNz0AMmy3Z2xWXJ1lUJqO95w/0?wx_fmt=png)
- Artifactory  (Maven Private Repository)
![Artifacotry](https://mmbiz.qlogo.cn/mmbiz_png/5Ofd65QfQBCLAg5aiazhvYYa9OGibBoJzy2ibF80UeDDxhrdxPaYFDcW7FXq2BEkKYuD2WYgdYicPhbvooBC5Sib3Ew/0?wx_fmt=png)
- SCM System (Subversion / Gitlab)
![gitlab](https://mmbiz.qlogo.cn/mmbiz_png/5Ofd65QfQBCLAg5aiazhvYYa9OGibBoJzyDHuRF9fg47CjmplRwiaTe429zpyKfqKUMgfHibVKp7F2wRAO1eUNiaFXQ/0?wx_fmt=png)
- Jenkins
![Jenkins](https://mmbiz.qlogo.cn/mmbiz_png/5Ofd65QfQBCLAg5aiazhvYYa9OGibBoJzyPmNl24m6MKicS0ej37YDF3YuqAhH8JGkANn7LnhWcqV98ltk6Cl9OHQ/0?wx_fmt=png)
- Sonarqube
![Sonarqube](https://mmbiz.qlogo.cn/mmbiz_png/5Ofd65QfQBCLAg5aiazhvYYa9OGibBoJzycia6UPyC304CoiarPUxtBoSaEwWicbISGVXRJw3ZGwXrrwFCicuUxpoVDw/0?wx_fmt=png)
- Redis
- RabbitMQ
- ELK (elasticsearch + logstash + kibana)
![logcenter](https://mmbiz.qlogo.cn/mmbiz_png/5Ofd65QfQBCLAg5aiazhvYYa9OGibBoJzyNRyICKQpjDTF0IbyMQvEum4olMnF5BrOBXHwxkjkuJArbP2vFowupw/0?wx_fmt=png)
- Monitoring ( Oracle Enterprise Manager / Promethues)
![Promethues](https://mmbiz.qlogo.cn/mmbiz_png/5Ofd65QfQBCLAg5aiazhvYYa9OGibBoJzy9sPUnDp5sb8ZkMKDk1GMKzZ2a1NvfU4RVg9pmMgk36ibmsPT9b2Xzdw/0?wx_fmt=png)
- Mysql Galara Cluster
- Spring Eureka
![Eureka](https://mmbiz.qlogo.cn/mmbiz_png/5Ofd65QfQBCLAg5aiazhvYYa9OGibBoJzyb3yEjNF8GlVplkyD6Nruox48XdY6icib33bgelWMXJuIgKkIYrfRKvCQ/0?wx_fmt=png)
- Zipkin
![Zipkin](https://mmbiz.qlogo.cn/mmbiz_png/5Ofd65QfQBCLAg5aiazhvYYa9OGibBoJzykHCfPz7LsuY7VaZiaySibcGVbicvlTyHlQRwf06xLAtM926EDoDdmqmow/0?wx_fmt=png)

## 平台架构及拓扑

![  OceanFrame](https://mmbiz.qlogo.cn/mmbiz_png/5Ofd65QfQBAK7QQJXOwPcvvOcnhG4hRJ2jhIYCEnBUaQhhpic8JUicJVaq7Lzmjsn6kRoib48oyQAXvhnjKjGdC4Q/0?wx_fmt=png)
*Diagram Author: CaryChen*

说明：

## 各组件安装说明

未来我们将整理Ocean平台各组件的安装说明，如下

   1. Kubernetes Installation Guide
   2. Vmware Harbor Installation Guide
   3. Jenkins Installation Guide
   4. Gitlab Installation Guide
   5. ELK Installation Guide
   6. Redis Installation Guide
   7. Mysql Galara Cluster Installation Guide
   8. Promethues Installation Guide
   9. Ceph Installation Guide
   10. Sonarqube Installation Guide
   11. RabbitMQ Installation Guide
   12. Artifactory Installation Guide

## 各组件使用说明

Ocean平台提供的服务有些向使用者开放服务，有些对服务使用者透明。 从服务使用者角度，提供如下使用说明：

> **开发过程使用Ocean指南**

- 开发过程中如何使用gitlab
- 开发过程中如何使用Jenkins
- 开发过程中如何使用日志框架将日志输出到集中日志平台
- 在Logcenter中如何查看及分析日志
- 开发过程如何使用Maven以及私有仓库Artifactory
- 开发如何使用S3接口使用Ceph存储

> **分布式开发指南**

- 如何使用Ocean平台上的配置中心(Spring Config Server)
- 如何使用Ocean平台上的服务注册/服务发现 （Spring Eureka Server Cluster)
- 如何使用服务跟踪 (Spring Zipkin)

> **从服务提供及维护者角度，提供如下使用说明**

1. Gitlab权限管理
2. Jenkins日常管理
3. ELK日志分析，ELK->Promethues日志报警
4. 私有云Mysql Galara Cluster Database维护指南
5. Spring 配置中心及服务注册发现中心的监控及维护
6. Maven私有仓库的管理
7. 推广培训
