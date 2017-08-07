# <font color="#003399">Ocean容器云平台上的Spring Cloud微服务Demo实践</font>

--------------------------------------------------------------------------------

## Ocean容器云平台介绍

![  OceanFrame](https://mmbiz.qlogo.cn/mmbiz_png/5Ofd65QfQBAK7QQJXOwPcvvOcnhG4hRJ2jhIYCEnBUaQhhpic8JUicJVaq7Lzmjsn6kRoib48oyQAXvhnjKjGdC4Q/0?wx_fmt=png)

容器平台组件：

Seq | Component Name      | URL Address
--- | ------------------- | -------------------------------------
1   | Jenkins             | <http://jenkins-edge.aegonthtf.com>
2   | Gitlab              | <http://gitlab.aegonthtf.com>
3   | LogCenter           | <http://logcenter-dev.aegonthtf.com>
4   | Promethues          | <http://grafana.aegonthtf.com>
5   | k8s dev             | <http://k8s-dev-backup.aegonthtf.com>
6   | weave scope monitor | <http://10.72.243.137:32358>

演示Case：

1. 使用Spring框架编写一个简单示例程序，程序源码保存至gitlab, 使用同方全球持续集成部署Jenkins Pipeline部署至k8s容器平台。
2. 演示和讨论Actuator对应用容器化以及系统后期监控和运维的重要性。
3. 在k8s上扩展/缩小部署实例数，演示容器平台实现灵活的容量变化。
4. 示例程序实现日志集中保存，日志输往集中日志平台。 演示在集中日志平台查询日志。
5. 演示在集中日志平台查询示例程序的访问统计。
6. 简单演示Promethues/WeaveScope监控Redis以及k8s Platform的情况。
7. 讨论。

## Spring Cloud介绍

![Spring Cloud Architecture](images/springCloudArchitecture.png)

Spring Cloud各组件地址：

Seq | Component Name            | URL Address
--- | ------------------------- | ---------------------------------
1   | Config Center             | <http://earthcore.aegonthtf.com>
2   | Register/Discovery Center | <http://eureka-dev.aegonthtf.com>
3   | ZipKin                    | <http://zipkin-dev.aegonthtf.com>
4   | Turbine                   | <http://turbine.aegonthtf.com>
5   | API Gateway               | <http://zuul.aegonthtf.com>

## Spring Cloud Demo

演示Case：

基于Spring Cloud框架开发有三个服务，1. Send Message  2. Mail Notifier 3. Page Notifier
三个服务均注册在服务注册中心，每个服务至少有两个以上的实例，每个实例的配置信息均保存在配置中心。三个服务均以Zuul API的方式暴露服务。
三个服务没有任何功能，仅仅是记录日志。 Send Message调用Mail Notifier和Page Notifier模拟发出通知的功能。 其中Page Notifier每次被调用会消耗4s的时间，Mail Notifier会立即返回。

1. 介绍、演示、讨论Spring Cloud中的配置中心。
Config Server Gitlab Address: http://gitlab.aegonthtf.com/aegonthtf-config-center/app-config-center

![Config Server](images/configServer.png)
![ConfigServerUI](images/configserverui.png)
2. 介绍、演示、讨论Spring Cloud中的服务注册和发现中心。
![Eureka](images/eureka.png)
3. 介绍、演示、讨论Spring Cloud中的服务跟踪。
![ZipKin](images/zipkin-g.gif)
4. 介绍、演示、讨论Spring Cloud中的断路器及实时监控,Spring Cloud中的服务API网关
6. 介绍、演示、讨论Postman测试API
![service-access](images/service-access.gif)

7. 介绍、演示、讨论Apache Jemeter针对API进行压力测试。
![stress](images/stress.gif)
