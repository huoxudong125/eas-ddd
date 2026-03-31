# EAS-DDD 项目 Code Wiki

## 1. 项目概述

### 1.1 项目简介
EAS-DDD 是一个基于领域驱动设计(DDD)实战的企业级应用系统案例项目，为 GitChat 课程《领域驱动设计实战（战略篇）》与《领域驱动设计实战（战术篇）》提供的实战项目案例。该案例取材自真实项目，代码完全按照 DDD 的过程进行建模、设计与编码。

### 1.2 项目特点
- 采用完整的领域驱动设计方法
- 单体架构设计，但易于迁移至微服务
- 使用 MyBatis 作为持久化框架（同时也展示了 JPA 风格）
- 采用测试驱动开发(TDD)
- 支持 Flyway 数据库版本管理
- 完整的单元测试和集成测试覆盖

### 1.3 技术栈

| 组件 | 版本 | 说明 |
|------|------|------|
| Java | 8+ | 开发语言 |
| Spring Boot | 2.1.9 | 应用框架 |
| Spring Framework | 5.2.2+ | 核心框架 |
| MyBatis | 3.5.3 | ORM 框架 |
| Druid | 1.1.20 | 数据库连接池 |
| MySQL | 8.0 Community | 数据库 |
| Flyway | 6.0.4 | 数据库迁移工具 |
| Maven | 3+ | 构建工具 |
| JUnit | 4.12 | 单元测试框架 |
| Mockito | 1.10.19 | Mock 框架 |
| AssertJ | 3.6.2 | 断言库 |

## 2. 项目架构

### 2.1 整体架构
项目采用单体架构，遵循领域驱动设计的分层架构模式：

```
┌─────────────────────────────────────────────┐
│              用户界面层 (UI)                 │
├─────────────────────────────────────────────┤
│          北向网关层 (North Gateway)         │
│  - REST Resources   - Local App Services    │
├─────────────────────────────────────────────┤
│              领域层 (Domain)                 │
│  - Aggregates  - Entities  - Value Objects  │
│  - Domain Services - Domain Events           │
├─────────────────────────────────────────────┤
│          南向网关层 (South Gateway)         │
│  - Repository Ports  - External Ports       │
├─────────────────────────────────────────────┤
│          基础设施层 (Infrastructure)        │
│  - Repository Adapters  - External Adapters │
└─────────────────────────────────────────────┘
```

### 2.2 模块结构

```
eas-ddd/
├── ddd-core/              # DDD 核心框架模块
├── eas-employee/          # 员工限界上下文
├── eas-project/           # 项目限界上下文
├── eas-training/          # 培训限界上下文（主要业务模块）
├── eas-entry/             # 应用入口模块
└── pom.xml                # 父 POM
```

#### 2.2.1 ddd-core 模块
DDD 核心框架，提供通用的基础设施和领域抽象。

**主要功能：**
- 领域对象基类和接口
- 事件基础设施
- 异常体系
- 网关抽象
- 注解体系

#### 2.2.2 eas-training 模块
培训限界上下文，是项目的核心业务模块。

**主要功能：**
- 培训管理
- 培训票管理
- 提名流程
- 通知服务
- 学习记录

#### 2.2.3 eas-employee / eas-project 模块
员工和项目限界上下文模块（当前结构较简单）。

#### 2.2.4 eas-entry 模块
应用程序入口，包含 Spring Boot 主程序和配置。

## 3. 核心框架详解 (ddd-core)

### 3.1 领域层抽象

#### 3.1.1 Identity 接口体系

**接口定义：** [Identity](file:///workspace/ddd-core/src/main/java/xyz/zhangyi/ddd/core/domain/Identity.java)

**抽象基类：** [AbstractIdentity](file:///workspace/ddd-core/src/main/java/xyz/zhangyi/ddd/core/domain/AbstractIdentity.java)

```java
public abstract class AbstractIdentity<T> implements Identity<T> {
    private T value;
    
    protected AbstractIdentity(T value) {
        this.setId(value);
    }
    
    @Override
    public T value() {
        return value;
    }
    
    // 包含值验证、equals、hashCode 等通用实现
}
```

**特点：**
- 强制 ID 值非空
- 支持自定义值验证
- 提供完整的 equals/hashCode 实现
- 类型安全的 ID 封装

### 3.2 注解体系

#### 3.2.1 领域注解

| 注解 | 位置 | 说明 |
|------|------|------|
| [@Aggregate](file:///workspace/ddd-core/src/main/java/xyz/zhangyi/ddd/core/stereotype/Aggregate.java) | 类 | 标识聚合根 |
| [@DomainService](file:///workspace/ddd-core/src/main/java/xyz/zhangyi/ddd/core/stereotype/DomainService.java) | 类 | 标识领域服务 |
| [@BoundedContext](file:///workspace/ddd-core/src/main/java/xyz/zhangyi/ddd/core/stereotype/BoundedContext.java) | 包/类 | 标识限界上下文 |
| [@SubDomain](file:///workspace/ddd-core/src/main/java/xyz/zhangyi/ddd/core/stereotype/SubDomain.java) | 包/类 | 标识子域 |

#### 3.2.2 网关注解

| 注解 | 位置 | 说明 |
|------|------|------|
| [@Port](file:///workspace/ddd-core/src/main/java/xyz/zhangyi/ddd/core/stereotype/Port.java) | 接口 | 标识端口 |
| [@Adapter](file:///workspace/ddd-core/src/main/java/xyz/zhangyi/ddd/core/stereotype/Adapter.java) | 类 | 标识适配器 |
| [@Remote](file:///workspace/ddd-core/src/main/java/xyz/zhangyi/ddd/core/stereotype/Remote.java) | 类 | 标识远程资源 |
| [@Local](file:///workspace/ddd-core/src/main/java/xyz/zhangyi/ddd/core/stereotype/Local.java) | 类 | 标识本地服务 |
| [@MessageContract](file:///workspace/ddd-core/src/main/java/xyz/zhangyi/ddd/core/stereotype/MessageContract.java) | 类 | 标识消息契约 |

### 3.3 事件基础设施

#### 3.3.1 事件接口

- [Event](file:///workspace/ddd-core/src/main/java/xyz/zhangyi/ddd/core/event/Event.java) - 领域事件基础接口
- [ApplicationEvent](file:///workspace/ddd-core/src/main/java/xyz/zhangyi/ddd/core/event/ApplicationEvent.java) - 应用事件

#### 3.3.2 事件发布基础设施

- [EventPublisher](file:///workspace/ddd-core/src/main/java/xyz/zhangyi/ddd/core/gateway/south/port/EventPublisher.java) - 事件发布端口
- [EventKafkaPublisher](file:///workspace/ddd-core/src/main/java/xyz/zhangyi/ddd/core/gateway/south/adapter/EventKafkaPublisher.java) - Kafka 事件发布适配器

### 3.4 异常体系

项目定义了完整的异常层级结构：

```
ApplicationException (应用异常基类)
├── ApplicationDomainException (领域异常)
├── ApplicationInfrastructureException (基础设施异常)
└── ApplicationValidationException (验证异常)

DomainException (领域异常基类)
```

相关文件：
- [ApplicationException](file:///workspace/ddd-core/src/main/java/xyz/zhangyi/ddd/core/exception/ApplicationException.java)
- [DomainException](file:///workspace/ddd-core/src/main/java/xyz/zhangyi/ddd/core/exception/DomainException.java)

### 3.5 网关基础设施

#### 3.5.1 北向网关
- [Resources](file:///workspace/ddd-core/src/main/java/xyz/zhangyi/ddd/core/gateway/north/Resources.java) - REST 资源辅助类，提供统一的响应处理

## 4. 培训限界上下文详解 (eas-training)

### 4.1 模块架构

```
trainingcontext/
├── domain/                      # 领域层
│   ├── candidate/              # 候选人聚合
│   ├── course/                 # 课程相关
│   ├── exception/              # 领域异常
│   ├── learning/               # 学习聚合
│   ├── notification/           # 通知服务
│   ├── ticket/                 # 培训票聚合（核心）
│   ├── tickethistory/          # 票历史记录
│   ├── training/               # 培训聚合
│   └── validate/               # 有效期验证
├── north/                       # 北向网关
│   ├── local/appservice/       # 应用服务
│   ├── remote/resource/        # REST 资源
│   └── message/                # 消息契约
└── south/                       # 南向网关
    ├── adapter/                # 适配器实现
    │   ├── client/             # 外部客户端适配器
    │   └── repository/         # 仓储适配器
    └── port/                   # 端口定义
        ├── client/             # 外部客户端端口
        └── repository/         # 仓储端口
```

### 4.2 核心领域模型

#### 4.2.1 培训聚合

**聚合根：** [Training](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/training/Training.java)

**职责：** 表示一个培训课程实例

**核心属性：**
- `id`: [TrainingId](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/training/TrainingId.java) - 培训ID
- `title`: 培训标题
- `description`: 培训描述
- `beginTime` / `endTime`: 培训开始/结束时间
- `place`: 培训地点
- `courseId`: [CourseId](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/course/CourseId.java) - 关联课程ID

#### 4.2.2 培训票聚合

**聚合根：** [Ticket](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/ticket/Ticket.java)

**职责：** 管理培训票的生命周期和提名流程

**核心属性：**
- `id`: [TicketId](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/ticket/TicketId.java) - 票ID
- `trainingId`: 关联培训ID
- `ticketStatus`: [TicketStatus](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/ticket/TicketStatus.java) - 票状态
- `nomineeId`: 被提名人ID

**核心业务方法：**

```java
// 提名候选人
public TicketHistory nominate(Candidate candidate, Nominator nominator) {
    validateTicketStatus();      // 验证票状态（必须是可用的）
    doNomination(candidate);      // 执行提名
    return generateHistory(candidate, nominator);  // 生成历史记录
}
```

**票状态机：**

| 状态 | 说明 |
|------|------|
| Available | 可用（可被提名）|
| WaitForConfirm | 待确认 |
| Confirmed | 已确认 |
| Rejected | 已拒绝 |

#### 4.2.3 票历史记录

**实体：** [TicketHistory](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/tickethistory/TicketHistory.java)

**职责：** 记录票的每次状态变更

**核心属性：**
- `ticketId`: 关联票ID
- `ticketOwner`: [TicketOwner](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/tickethistory/TicketOwner.java) - 票拥有者
- `stateTransit`: [StateTransit](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/tickethistory/StateTransit.java) - 状态转换
- `operationType`: [OperationType](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/tickethistory/OperationType.java) - 操作类型
- `operator`: [Operator](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/tickethistory/Operator.java) - 操作者
- `operatedAt`: 操作时间

#### 4.2.4 候选人

**实体：** [Candidate](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/candidate/Candidate.java)

**职责：** 表示可以被提名参加培训的员工

#### 4.2.5 通知服务

**领域服务：** [NotificationService](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/notification/NotificationService.java)

**职责：** 处理培训相关的通知发送

**相关组件：**
- [MailTemplate](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/notification/MailTemplate.java) - 邮件模板
- [TemplateType](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/notification/TemplateType.java) - 模板类型
- [NominationNotificationComposer](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/notification/NominationNotificationComposer.java) - 提名通知组装器

#### 4.2.6 学习记录

**聚合根：** [Learning](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/learning/Learning.java)

**职责：** 记录员工的学习情况

### 4.3 领域服务

| 服务 | 职责 |
|------|------|
| [TrainingService](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/training/TrainingService.java) | 培训相关业务逻辑 |
| [TicketService](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/ticket/TicketService.java) | 培训票相关业务逻辑 |
| [NominationService](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/ticket/NominationService.java) | 提名流程业务逻辑 |
| [LearningService](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/learning/LearningService.java) | 学习相关业务逻辑 |
| [NotificationService](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/notification/NotificationService.java) | 通知发送业务逻辑 |

### 4.4 应用服务

| 服务 | 职责 |
|------|------|
| [TrainingAppService](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/north/local/appservice/TrainingAppService.java) | 培训应用服务 |
| [NominationAppService](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/north/local/appservice/NominationAppService.java) | 提名应用服务 |

### 4.5 REST 资源

| 资源 | 路径 | 说明 |
|------|------|------|
| [TrainingResource](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/north/remote/resource/TrainingResource.java) | /trainings | 培训资源 |
| [TicketResource](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/north/remote/resource/TicketResource.java) | /tickets | 培训票资源 |

### 4.6 仓储端口

| 仓储 | 职责 |
|------|------|
| [TrainingRepository](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/south/port/repository/TrainingRepository.java) | 培训聚合仓储 |
| [TicketRepository](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/south/port/repository/TicketRepository.java) | 培训票聚合仓储 |
| [TicketHistoryRepository](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/south/port/repository/TicketHistoryRepository.java) | 票历史仓储 |
| [CandidateRepository](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/south/port/repository/CandidateRepository.java) | 候选人仓储 |
| [LearningRepository](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/south/port/repository/LearningRepository.java) | 学习记录仓储 |
| [MailTemplateRepository](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/south/port/repository/MailTemplateRepository.java) | 邮件模板仓储 |
| [ValidDateRepository](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/south/port/repository/ValidDateRepository.java) | 有效期仓储 |

### 4.7 外部端口

| 端口 | 职责 |
|------|------|
| [NotificationClient](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/south/port/client/NotificationClient.java) | 通知客户端端口 |

**适配器实现：** [NotificationClientAdapter](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/south/adapter/client/NotificationClientAdapter.java)

### 4.8 MyBatis 类型处理器

为了支持自定义值对象的持久化，项目实现了类型处理器：

- [TicketIdTypeHandler](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/south/adapter/repository/typehandler/TicketIdTypeHandler.java)
- [TrainingIdTypeHandler](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/south/adapter/repository/typehandler/TrainingIdTypeHandler.java)
- [CourseIdTypeHandler](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/south/adapter/repository/typehandler/CourseIdTypeHandler.java)

## 5. 数据库设计

### 5.1 数据库迁移脚本

项目使用 Flyway 进行数据库版本管理，迁移脚本位于 [eas-training/src/main/resources/db/migration/](file:///workspace/eas-training/src/main/resources/db/migration/)

**V1__create_training_and_ticket_table.sql** - 创建基础表结构
**V2__insert_test_data.sql** - 插入测试数据

### 5.2 核心数据表

#### 5.2.1 training 表
存储培训信息

| 字段 | 类型 | 说明 |
|------|------|------|
| id | VARCHAR(50) | 主键 |
| title | VARCHAR(50) | 培训标题 |
| description | VARCHAR(200) | 培训描述 |
| beginTime | TIMESTAMP | 开始时间 |
| endTime | TIMESTAMP | 结束时间 |
| place | VARCHAR(200) | 培训地点 |
| courseId | VARCHAR(50) | 课程ID |

#### 5.2.2 ticket 表
存储培训票信息

| 字段 | 类型 | 说明 |
|------|------|------|
| id | VARCHAR(50) | 主键 |
| ticketStatus | VARCHAR(20) | 票状态 |
| trainingId | VARCHAR(50) | 关联培训ID |
| nomineeId | VARCHAR(50) | 被提名人ID |

#### 5.2.3 ticket_history 表
存储票的历史变更记录

| 字段 | 类型 | 说明 |
|------|------|------|
| id | VARCHAR(50) | 主键 |
| ticketId | VARCHAR(50) | 关联票ID |
| ownerId | VARCHAR(50) | 拥有者ID |
| ownerType | VARCHAR(20) | 拥有者类型 |
| fromStatus | VARCHAR(20) | 源状态 |
| toStatus | VARCHAR(20) | 目标状态 |
| OperationType | VARCHAR(20) | 操作类型 |
| operatorId | VARCHAR(50) | 操作者ID |
| operatorName | VARCHAR(20) | 操作者姓名 |
| operatedAt | TIMESTAMP | 操作时间 |

#### 5.2.4 其他表
- **learning** - 学习记录表
- **candidate** - 候选人表
- **valid_date** - 有效期表
- **mail_template** - 邮件模板表

## 6. 模块依赖关系

### 6.1 Maven 模块依赖

```
eas-entry (入口模块)
├── ddd-core
├── eas-employee
├── eas-project
└── eas-training

eas-training
└── ddd-core

eas-employee
└── ddd-core

eas-project
└── ddd-core

ddd-core
(无内部依赖)
```

### 6.2 关键第三方依赖

| 依赖 | 用途 |
|------|------|
| spring-boot-starter-web | Web 应用支持 |
| spring-boot-starter-data-jpa | JPA 支持 |
| mybatis-spring-boot-starter | MyBatis 集成 |
| mybatis-typehandlers-jsr310 | Java 8 时间类型支持 |
| druid | 数据库连接池 |
| mysql-connector-java | MySQL 驱动 |
| guava | Google 工具库 |
| fastjson | JSON 处理 |
| kafka-clients | Kafka 客户端 |
| ST4 | 模板引擎 |
| junit / mockito / assertj | 测试框架 |

## 7. 项目运行方式

### 7.1 环境要求

- Java 8+
- Maven 3+
- MySQL 8.0 Community

### 7.2 数据库准备

**1. 创建数据库**
```sql
CREATE DATABASE `eas-db` CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

**2. 配置数据库连接**

修改根 [pom.xml](file:///workspace/pom.xml) 中的数据库配置：
```xml
<db.driver>com.mysql.jdbc.Driver</db.driver>
<db.url>jdbc:mysql://localhost:3306/eas-db?serverTimezone=UTC</db.url>
<db.username>sa</db.username>
<db.password>123456</db.password>
```

**3. 执行数据库迁移**
```bash
# 清理数据库
mvn package flyway:clean

# 执行迁移
mvn package flyway:migrate
```

### 7.3 测试运行

**运行单元测试**
```bash
mvn test
```

**运行集成测试（需要数据库）**
```bash
mvn integration-test
```

**跳过测试**
```bash
mvn package -DskipTests
```

### 7.4 启动应用

**方式一：IDE 运行**
直接运行 [EasApplication](file:///workspace/eas-entry/src/main/java/xyz/zhangyi/ddd/eas/EasApplication.java) 的 main 方法

**方式二：Maven 运行**
```bash
cd eas-entry
mvn spring-boot:run
```

**应用配置**

配置文件位于 [eas-entry/src/main/resources/application.yml](file:///workspace/eas-entry/src/main/resources/application.yml)

默认端口：8080

## 8. 关键设计模式与原则

### 8.1 领域驱动设计模式

| 模式 | 应用示例 |
|------|----------|
| 聚合 (Aggregate) | Training, Ticket |
| 值对象 (Value Object) | TicketId, TrainingId, StateTransit |
| 领域服务 (Domain Service) | TicketService, NominationService |
| 仓储 (Repository) | TicketRepository, TrainingRepository |
| 领域事件 (Domain Event) | Event 接口体系 |
| 防腐层 (ACL) | 端口-适配器模式 |

### 8.2 架构模式

| 模式 | 应用 |
|------|------|
| 分层架构 | UI / 应用 / 领域 / 基础设施 |
| 端口-适配器 (Hexagonal) | 南向网关的 Port/Adapter 分离 |
| CQRS（概念上）| 读写职责分离 |

### 8.3 设计原则

- **单一职责原则 (SRP)**: 每个类职责明确
- **开闭原则 (OCP)**: 通过接口和抽象实现扩展
- **里氏替换原则 (LSP)**: 继承体系设计合理
- **接口隔离原则 (ISP)**: 接口定义精简
- **依赖倒置原则 (DIP)**: 依赖抽象而非具体实现

## 9. 测试策略

### 9.1 测试分类

| 测试类型 | 命名规范 | 运行命令 |
|----------|----------|----------|
| 单元测试 | *Test.java | mvn test |
| 集成测试 | *IT.java | mvn integration-test |

### 9.2 测试覆盖范围

- **领域层**: 高覆盖率的单元测试
- **应用服务**: 集成测试
- **资源层**: 集成测试

## 10. 扩展与演化

### 10.1 向微服务迁移

当前采用单体架构，但可轻松迁移至微服务：

1. 为每个限界上下文模块添加 spring-boot-starter-web 依赖
2. 配置独立的数据库
3. 独立部署每个服务
4. 添加服务间通信机制（REST / 消息）

### 10.2 切换持久化框架

项目展示了 MyBatis 的用法，作者也提供了使用 JPA 的参考项目 [Payroll-DDD](https://github.com/agiledon/payroll-ddd)。

## 11. 参考资源

- 项目 Wiki: https://github.com/agiledon/eas-ddd/wiki
- GitChat 课程:
  - 《领域驱动设计实战（战略篇）》
  - 《领域驱动设计实战（战术篇）》
- 项目 Issues: https://github.com/agiledon/eas-ddd/issues
- JPA 版本参考: https://github.com/agiledon/payroll-ddd

## 12. 项目文件速查表

### 核心框架
- [pom.xml](file:///workspace/pom.xml) - 父 POM 配置
- [ddd-core/pom.xml](file:///workspace/ddd-core/pom.xml) - 核心框架 POM
- [AbstractIdentity.java](file:///workspace/ddd-core/src/main/java/xyz/zhangyi/ddd/core/domain/AbstractIdentity.java) - ID 基类
- [Aggregate.java](file:///workspace/ddd-core/src/main/java/xyz/zhangyi/ddd/core/stereotype/Aggregate.java) - 聚合注解

### 培训上下文
- [eas-training/pom.xml](file:///workspace/eas-training/pom.xml) - 培训模块 POM
- [Ticket.java](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/ticket/Ticket.java) - 培训票聚合
- [Training.java](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/training/Training.java) - 培训聚合
- [TicketRepository.java](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/south/port/repository/TicketRepository.java) - 票仓储
- [TrainingResource.java](file:///workspace/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/north/remote/resource/TrainingResource.java) - 培训资源
- [V1__create_training_and_ticket_table.sql](file:///workspace/eas-training/src/main/resources/db/migration/V1__create_training_and_ticket_table.sql) - 数据库表结构

### 应用入口
- [eas-entry/pom.xml](file:///workspace/eas-entry/pom.xml) - 入口模块 POM
- [EasApplication.java](file:///workspace/eas-entry/src/main/java/xyz/zhangyi/ddd/eas/EasApplication.java) - Spring Boot 主类
- [application.yml](file:///workspace/eas-entry/src/main/resources/application.yml) - 应用配置
