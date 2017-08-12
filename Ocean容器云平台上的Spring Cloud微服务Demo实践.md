# <font color="#003399">Ocean容器云平台上的Spring Cloud微服务Demo实践</font>

--------------------------------------------------------------------------------

## 讨论会议目标

1. 简要介绍同方全球下一代业务系统平台-Ocean容器平台的情况和架构
2. 讨论下一代业务系统平台给应用可能带来的便利和可能性

  - 以可接受的成本支持分布式应用的开发
  - 以容器的方式实现开发环境、测试环境、生产环境的应用计算实例的高度一致的可能性 计算和配置解耦，环境与应用的整体打包流转帮助我们实现如上目标
  - 日志信息的统一收集、保存、审计、分析
  - 计算实例的灵活扩缩能力，控制管理应用的部署成本和承载能力。
  - 应用容器实例的健康自我检测和故障自愈能力。

3. 对基于spring boot以及spring cloud实现的两个实例进行讨论，了解上述目标实现的情况。

## Ocean容器云平台架构概要介绍

![  OceanFrame](https://mmbiz.qlogo.cn/mmbiz_png/5Ofd65QfQBAK7QQJXOwPcvvOcnhG4hRJ2jhIYCEnBUaQhhpic8JUicJVaq7Lzmjsn6kRoib48oyQAXvhnjKjGdC4Q/0?wx_fmt=png)

容器平台组件：

seq | Component Name                   | URL Address
:-- | :------------------------------- | :---------------------------------------------------------------------------
1   | Jenkins                          | <http://jenkins-edge.aegonthtf.com>
2   | Gitlab                           | <http://gitlab.aegonthtf.com>
3   | LogCenter                        | <http://logcenter-dev.aegonthtf.com>
6   | Weave Scope Monitor              | <http://10.72.243.137:32358>
7   | Maven Private Repository         | <http://repository.aegonthtf.com>
8   | Harbor Registry                  | <http://registry.aegonthtf.com>
9   | SonarQube - Code Quality Control | <http://sonarqube-dev.aegonthtf.com>
4   | Promethues                       | <http://grafana.aegonthtf.com>
5   | K8s Dashboard                    | <http://k8s-dev-primary.aegonthtf.com> <http://k8s-dev-backup.aegonthtf.com>

## 讨论案例：

## spring boot demo演示

### 程序说明：

- 使用Spring Boot编写的demo应用
- 使用Sprng Cloud Config Server中央配置中心保存配置项
- 日志使用logback框架，使用中央配置中心的统一配置，根据部署环境的不同自动激活不同的日志设置，程序日志输往集中日志中心
- 程序仅仅有简单的功能，显示两个配置项的内容
- Source Code保存在同方Gitlab上.
- Demo程序使用Maven作为构建工具，使用同方私有Artifactory Maven仓库。
- 使用持续集成部署Jenkins Pipeline部署至k8s容器平台。

### 演示步骤

- 创建应用项目Spring Boot Demo
- Gitlab上创建仓库，将spring boot demo项目push到 Gitlab
- 为Spring boot demo项目配置两个Jenkins Pipeline Item,分别模拟测试环境和生产环境的发布过程
- 构建应用发布到两个模拟环境

  - 配置Jenkins Pipeline Item, 构建、质量检查、部署发布（模拟测试环境）
  - 配置Jenkins Pipeline Item, 构建、质量检查、部署发布（模拟生产环境） -

- 访问验证

  - 访问模拟测试环境<http://spring-boot-demo.aegonthtf.com/show显示测试配置项内容>
  - 访问模拟生产环境<http://spring-boot-demo-prod.aegonthtf.com/show显示生产配置项内容>
  - 分别在测试和生产日志平台上查看部署和访问产生的应用日志
  - 通过查看域名转发的access日志了解应用的访问日志、、

- 演示动态更改配置项并生效。在不停应用服务的情况下更改配置项。

- 简单讨论Spring Actuator模块的作用。（自动增加的运维心脏，健康监控探针）

  - 访问<http://spring-boot-demo.aegonthtf.com/env> 查看运行环境
  - 访问<http://spring-boot-demo.aegonthtf.com/health> 查看应用健康状况。可做监控探针，也是每个容器实例自我监控，发现不健康时自杀后重生的可用工具

- 演示一下在容器平台上灵活扩展／缩小应用实例数目达到灵活改变处理能力的操作。
- 示例程序实现日志集中保存，日志输往集中日志平台。 演示在集中日志平台查询日志。
- 演示在集中日志平台查询示例程序的访问统计。
- 简单演示Promethues/WeaveScope监控Redis以及k8s Platform的情况。
- 讨论。

## Spring Cloud介绍

![Spring Cloud Architecture](images/springCloudArchitecture.png)

Spring Cloud各组件地址：

Seq | Component Name             | URL Address
:-- | :------------------------- | :--------------------------------
1   | Spring Cloud Config Center | <http://earthcore.aegonthtf.com>
2   | Spring Cloud Eureka Center | <http://eureka-dev.aegonthtf.com>
3   | Spring Cloud ZipKin        | <http://zipkin-dev.aegonthtf.com>
4   | Spring Cloud Turbine       | <http://turbine.aegonthtf.com>
5   | Spring Cloud Zuul API Gateway      | <http://zuul.aegonthtf.com>

## Spring Cloud Demo

演示Case：

基于Spring Cloud框架开发有三个服务，1\. Send Message 2\. Mail Notifier 3\. Page Notifier 三个服务均注册在服务注册中心，每个服务至少有两个以上的实例，每个实例的配置信息均保存在配置中心。三个服务均以Zuul API的方式暴露服务。 三个服务没有任何功能，仅仅是记录日志。 Send Message调用Mail Notifier和Page Notifier模拟发出通知的功能。 其中Page Notifier每次被调用会消耗4s的时间，Mail Notifier会立即返回。

1. 介绍、演示、讨论Spring Cloud中的配置中心。

![Config Server](images/configServer.png) ![ConfigServerUI](images/configserverui.png)

1. 介绍、演示、讨论Spring Cloud中的服务注册和发现中心。 ![Eureka](images/eureka.png)
2. 介绍、演示、讨论Spring Cloud中的服务跟踪。 ![ZipKin](images/zipkin-g.gif)
3. 介绍、演示、讨论Spring Cloud中的断路器及实时监控,Spring Cloud中的服务API网关
4. 介绍、演示、讨论利用Postman测试API ![service-access](images/service-access.gif)

5. 介绍、演示、讨论Apache Jemeter针对API进行压力测试。 ![stress](images/stress.gif)
