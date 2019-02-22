# microservice-demo
《Spring microservices IN ACTION》一书项目实战

## 项目描述
本项目模拟一个名为 EagleEye 的产品，该产品是一个企业级软件资产管理应用程序，目标是使组织获得准确时间点的软件资产的描述。

基本业务功能有：库存、软件交付、许可证管理、合规、成本以及资本管理

## 项目笔记

### Sprig Cloud 配置服务器

依赖于spring-cloud-starter-config 这个库，在配置服务的application.yml中配置端口以及配置数据的访问路径

在引导类中使用 @EnableConfigServer注解使服务成为 Spring Cloud Config 服务，启动该服务就可以通过HTTP访问到配置数据了

在服务启动类使用spring boot actuator 的@RefreshScope注解，每当所属的配置服务器发生更新时，本服务可以通过POST访问 /refresh ,不用重启就能导入新的配置

#### 如何保护敏感信息？
Spring Cloud Config支持使用对称加密（共享秘钥）和非对称（公钥、私钥）

对称加密具体步骤：
（1）安装Oracle JCE jar 
（2）创建加密秘钥
（3）加密和解密属性
（4）配置微服务以在客户端中使用加密

对称加密 解密具体步骤：
（1）配置Spring Cloud Config不要在服务器端解密属性
（2）在其他服务上设置对称秘钥
（3）将spring-security-rsa 添加到需要解密服务的的pom.xml中

### 服务发现
- 服务发现模式用于抽象服务的物理位置
- 像Eureka这样的服务发现引擎可以在不影响服务客户端的情况下，无缝地向环境中添加和移除服务实例
- 通过在进行服务调用的客户端中缓存服务的物理位置，客户端负载均衡可以提供额外的性能和弹性
- Eureka是Netflix项目，在与Spring Cloud一起使用时，很容易对Eureka进行建立和配置


#### 构建Eureka服务

Spring Eureka依赖于 spring-cloud-starter-eureka-server 这个库

Eureka服务的几个重要配置：
server.port: 8761       -- 设置Eureka服务的默认端口
eureka.client.registerWithEureka: false     --在Spring Boot Eureka应用程序启动时不要通过Euraka注册，因为它本身就是Eureka服务
eureka.client.fetchRegistry: false      --不要在本地缓存注册表信息

在启动类添加 @EnableEurekaServer 就可以让这个服务成为一个Eureka服务

#### 使用 Spring Eureka 注册服务

依赖于 spring-cloud-starter-eureka-server 这个库

客户端的服务与Eureka通信的几个配置：
eureka.instance.preferIpAddress: true   -- 首选注册服务的IP，而不是服务器名称
eureka.client.registerWithEureka: true     -- 向Eureka注册服务
eureka.client.fetchRegistry: true           -- 让客户端的服务从本地缓存注册表中获取服务，而不是每次查找服务都调用Eureka服务。每个30秒，客户端就会重新联系Eureka服务，保证本地缓存是实时有效的
eureka.client.uri: http://localhost:8888        -- Eureka服务的地址列表，多个Eureka服务器就用 "," 隔开


可以使用Eureka 的 REST API 来查看服务注册表的内容：
http://<eureka server>:8761/eureka/apps/<APPID>

#### 使用服务发现来查找服务

这本书主要研究了三种不同的 Spring/Netflix 客户端库，服务消费者可以使用它们和Ribbon进行交互，调用服务
- Spring DiscoveryClient
- 启用了RestTemplate 的Spring DiscoveryClient
- Netflix Feign 客户端

### 客户端弹性模式

客户端弹性模式的重点是：在远处服务发生错误或表现不佳时保护远程资源(另一个微服务调用或数据库查询)的客户端免于崩溃

有四种客户端弹性模式：
- 客户端负载均衡模式 ：服务客户端缓存在服务发现期间检索到的微服务端点
- 断路器模式 ：断路器模式确保服务客户端不会重复调用失败的服务
- 后备模式  ：当调用失败时，后备模式询问是否有可执行的替代方案
- 舱壁模式  ：舱壁模式隔离服务客户端上不同的服务调用，以确保表现不佳的服务不会耗尽客户端的资源

#### Hystrix

依赖于 spring-cloud-starter-hystrix 库

启动项添加 @EnableCircuitBreaker

##### 使用Hystrix 实现断路器

对类方法使用@HystrixCommand ，标记为由Hystrix断路器进行管理

#### Hystrix 实现后备处理
在@HystrixCommand注解中添加 fallbackMethod(""<方法名>")，后备方法的返回值和参数要与原方法一致

#### Hystrix 实现舱壁模式
使用@HystrixCommand注解的其他属性：
（1）为方法调用建立一个单独线程池
（2）设置线程池中的线程数
（3）设置单个线程繁忙时可排队的请求数的队列大小

#### Hystrix 如何确定断路器何时跳闸？
每当Hystrix命令遇到服务错误时，它将开启一个10s的计时器，用于检查调用失败的频率

Hystrix做的第一件事就是查看10s内发生的调用数量    （如果调用次数少于在这个窗口内需要发生的最小调用次数，Hystrix不会采取行动）

如果超过错误阈值的百分比，Hystrix将断路器跳闸，防止更多的远程资源调用

跳闸时，Hystrix会尝试启动一个新的活动窗口，每隔5s，Hystrix会让一个调用通过，如果成功，Hystrix重置断路器让所有调用通过

如果失败，Hystrix将保持断路器断开，并在另一个5s里再次尝试上述步骤

#### HystrixConcurrentStrategy

Hystrix允许开发人员定义一种自定义的并发策略

实现自定义HystrixConcurrentStrategy 步骤：
(1)定义自定义的Hystrix并发策略类
(2)定义一个Callable类，将UserContext注入Hystrix中
(3)配置Spring Cloud以使用自定义Hystrix并发策略





