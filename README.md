# microservice-demo
《Spring microservices IN ACTION》一书项目实战

## 项目描述
本项目模拟一个名为 EagleEye 的产品，该产品是一个企业级软件资产管理应用程序，目标是使组织获得准确时间点的软件资产的描述。

基本业务功能有：库存、软件交付、许可证管理、合规、成本以及资本管理

## 项目笔记

### Sprig Cloud 分布式配置

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

#### 服务发现小结
- 服务发现模式用于抽象服务的物理位置
- Eureka可以在不影响服务客户端的情况下，无缝地向环境中添加和移除服务实例
- 通过在进行服务调用的客户端中缓存服务的物理位置，客户端负载均衡可以提供额外的性能和弹性
- 核心3种不同的机制调用服务
    - 使用Spring Cloud服务DiscoveryClient

    - 使用Spring Cloud和支持Ribbon的RestTemplate

    - 使用Spring Cloud和Netflix的Feign客户端

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

#### 客户端弹性模式小结
- 一个性能不佳的服务可能会引起资源耗尽的连锁反应，因为调用客户端中的线程被阻塞，以等待服务完成
- 3种核心客户端弹性模式： 断路器模式、后备模式、舱壁模式
- 断路器模式试图杀死运行缓慢和降级的系统调用，这样调用就会快速失败，并防止资源耗尽
- 后备模式通过对远程服务的调用失败或断路器跳闸的情况下，定义代替代码路径
- 舱壁模式通过对远程服务的调用隔离到它们自己的线程池中，使远程资源调用彼此分离。就算一组服务调用失败，这些失败也不会导致应用程序容器中的所有资源耗尽
- Hystrix库是高度可配置的，可以在全局、类和线程池级别设置
- Hystrix支持两种隔离模型： THREAD 、 SEMAPHORE
- Hystrix默认隔离模型THREAD完全隔离Hystrix保护的调用，但不会将父进程的上下文传播到Hystrix管理的线程
- SEMAPHORE 不使用单独的线程进行Hystrix调用。虽然这更有效率，但如果Hystrix中断了调用，它也会让服务变得不可预测
- Hystrix允许通过自定义HystrixConcurrentStrategy实现，将父进程上下文注入Hystrix管理的线程中


### 服务网关
将一些横切关注点(安全、日志等)抽象成一个独立且作为应用程序中所有微服务调用的过滤器和路由器服务

服务客户端不直接调用服务，所有的服务调用都通过网关进行路由，然后被路由到最终目的地


#### Zuul服务 
依赖于 spring-cloud-starter-zuul 库

启动类加上 @EnableZuulProxy

##### Zuul服务中配置路由的三种方式
- 通过服务发现自动配置路由
- 使用服务发现手动映射路由
- 使用静态URL手动映射路由

##### Zuul过滤器

前置过滤器(HTTP请求到达实际服务之前对HTTP请求进行检查和修改)

后置过滤器(检查响应，可以修改相应和添加额外信息)

路由过滤器(可以为服务客户端添加智能路由)

#### 服务网关小结
- Zuul服务网关与Eureka集成，可以自动将通过Eureka注册的服务映射到Zuul路由
- Zuul可以对所有正在管理的路由添加前缀，因此轻松地给路由添加 /api 之类的前缀
- 可以使用Zuul手动定义路由映射，这些路由映射是在应用程序配置文件中手动配置的
- 通过使用Spring Cloud Config服务器，可以动态地重新加载路由映射，而无须重新启动Zuul服务器
- 可以在全局和个体服务水平上定制Hystrix和Ribbon的超时
- Zuul允许通过Zuul过滤器实现自定义业务逻辑
- Zuul前置过滤器可以用于生成一个关联ID，该关联ID可以注入流经Zuul的每个服务
- Zuul后置过滤器可以将关联ID注入服务客户端每个HTTP服务相应中
- 自定义Zuul路由过滤器可以根据Eureka服务ID执行动态路由
