# EagleEye 微服务架构项目

学习《Spring Microservices in Action》书籍时写的实战项目，实现了一个企业级软件资产管理应用程序EagleEye。该项目采用Spring Cloud微服务架构。

## 🏗️ 项目架构

EagleEye是一个企业级软件资产管理平台，提供以下核心功能：
- **库存管理** - 软件资产清单管理
- **许可证管理** - 软件许可证分配和跟踪
- **合规管理** - 许可证合规性检查
- **成本管理** - 软件资产成本分析
- **组织管理** - 企业组织架构管理

## 🚀 技术栈

- **Spring Boot** 1.4.4 - 微服务框架
- **Spring Cloud** Camden.SR5 - 微服务治理
- **Spring Cloud Config** - 分布式配置管理
- **Netflix Eureka** - 服务注册与发现
- **Netflix Zuul** - API网关
- **Netflix Hystrix** - 断路器模式
- **Spring Cloud Feign** - 声明式HTTP客户端
- **Spring Data JPA** - 数据访问层
- **MySQL** - 关系型数据库
- **Docker** - 容器化部署

## 🏛️ 微服务架构

### 服务组件

| 服务名称 | 端口 | 功能描述 |
|---------|------|----------|
| **Config Server** | 8888 | 分布式配置中心，统一管理所有微服务配置 |
| **Eureka Server** | 8761 | 服务注册中心，提供服务发现功能 |
| **Organization Service** | 8080 | 组织管理服务，管理企业组织信息 |
| **Licensing Service** | 8081 | 许可证管理服务，处理软件许可证相关业务 |
| **Zuul Gateway** | 5555 | API网关，统一入口和路由管理 |

### 架构特点

- **服务发现**: 基于Eureka实现服务注册与发现
- **配置管理**: 使用Spring Cloud Config实现集中化配置
- **API网关**: Zuul提供统一API入口和路由
- **容错处理**: Hystrix实现断路器、舱壁和后备模式
- **负载均衡**: Ribbon实现客户端负载均衡
- **服务调用**: 支持RestTemplate、Feign等多种调用方式

## 📁 项目结构

```
microservice-demo/
├── confsvr/                    # 配置服务
│   ├── src/main/java/
│   └── src/main/resources/config/
├── eurekasvr/                  # 服务注册中心
│   └── src/main/java/
├── organization-service/        # 组织管理服务
│   ├── src/main/java/
│   └── src/main/resources/
├── licensing-service/          # 许可证管理服务
│   ├── src/main/java/
│   └── src/main/resources/
└── zuulsvr/                    # API网关
    └── src/main/java/
```

## 🛠️ 快速开始

### 环境要求

- Java 8+
- Maven 3.6+
- MySQL 5.7+
- Docker (可选)

### 1. 数据库准备

创建MySQL数据库并执行SQL脚本：

```sql
-- 创建数据库
CREATE DATABASE eagle_eye_licensing;
CREATE DATABASE eagle_eye_organization;

-- 执行各服务的schema.sql文件
```

### 2. 启动服务

**按以下顺序启动服务：**

```bash
# 1. 启动配置服务
cd confsvr
mvn spring-boot:run

# 2. 启动Eureka服务
cd ../eurekasvr
mvn spring-boot:run

# 3. 启动组织服务
cd ../organization-service
mvn spring-boot:run

# 4. 启动许可证服务
cd ../licensing-service
mvn spring-boot:run

# 5. 启动Zuul网关
cd ../zuulsvr
mvn spring-boot:run
```

### 3. 验证服务

- **Eureka控制台**: http://localhost:8761
- **配置服务**: http://localhost:8888
- **组织服务**: http://localhost:8080
- **许可证服务**: http://localhost:8081
- **API网关**: http://localhost:5555

## 🐳 Docker部署

### 构建镜像

```bash
# 构建所有服务镜像
mvn clean package docker:build
```

### 运行容器

```bash
# 使用Docker Compose (需要创建docker-compose.yml)
docker-compose up -d
```

## 📚 API文档

### 组织服务 API

| 方法 | 路径 | 描述 |
|------|------|------|
| GET | `/v1/organizations/{id}` | 获取组织信息 |
| POST | `/v1/organizations` | 创建组织 |
| PUT | `/v1/organizations/{id}` | 更新组织 |
| DELETE | `/v1/organizations/{id}` | 删除组织 |

### 许可证服务 API

| 方法 | 路径 | 描述 |
|------|------|------|
| GET | `/v1/organizations/{orgId}/licenses` | 获取组织所有许可证 |
| GET | `/v1/organizations/{orgId}/licenses/{id}` | 获取特定许可证 |
| POST | `/v1/organizations/{orgId}/licenses` | 创建许可证 |
| PUT | `/v1/organizations/{orgId}/licenses/{id}` | 更新许可证 |
| DELETE | `/v1/organizations/{orgId}/licenses/{id}` | 删除许可证 |

### 通过网关访问

所有API都可通过Zuul网关访问，路径前缀为`/api`：

- 组织服务: `http://localhost:5555/api/organization/v1/organizations`
- 许可证服务: `http://localhost:5555/api/licensing/v1/organizations/{orgId}/licenses`

## 🔧 配置说明

### 服务端口配置

- Config Server: 8888
- Eureka Server: 8761
- Organization Service: 8080
- Licensing Service: 8081
- Zuul Gateway: 5555

### 数据库配置

各服务的数据库配置在对应的`application.yml`文件中：

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/eagle_eye_licensing
    username: your_username
    password: your_password
```

## 🎯 核心特性

### 1. 服务发现与注册
- 基于Eureka实现服务自动注册与发现
- 支持服务健康检查和故障转移

### 2. 配置管理
- 集中化配置管理
- 支持配置热更新
- 支持多环境配置

### 3. API网关
- 统一API入口
- 请求路由和负载均衡
- 请求过滤和转换

### 4. 容错处理
- 断路器模式防止级联故障
- 舱壁模式隔离资源
- 后备模式提供降级服务

### 5. 服务间通信
- 支持多种服务调用方式
- 客户端负载均衡
- 服务熔断和重试

## 📖 学习要点

这个项目展示了以下微服务开发的核心概念：

1. **微服务拆分** - 按业务领域拆分服务
2. **服务治理** - 服务注册、发现、配置管理
3. **API网关** - 统一入口和横切关注点
4. **容错设计** - 断路器、舱壁、后备模式
5. **服务通信** - 同步和异步通信模式
6. **数据管理** - 分布式数据一致性
7. **监控运维** - 服务监控和日志管理




---

**注意**: 这是一个学习项目，展示了微服务架构的基本实现。在生产环境中使用前，请确保进行充分的安全性和性能测试。