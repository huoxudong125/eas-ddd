# EAS-DDD 项目学习文档

## 1. 项目概述

EAS-DDD是一个完整的领域驱动设计(DDD)实战项目，为GitChat课程《领域驱动设计实战（战略篇）》与《领域驱动设计实战（战术篇）》提供的实战案例。

### 技术栈
- **Java 8+** - 开发语言
- **Spring Boot 2.1.9** - 应用框架
- **MyBatis 3.5.3** - 持久化框架
- **MySQL 8.0** - 数据库
- **Flyway 6.0.4** - 数据库迁移工具

### 核心模块
- **ddd-core** - DDD核心框架
- **eas-training** - 培训限界上下文（核心业务模块）
- **eas-employee** - 员工限界上下文
- **eas-project** - 项目限界上下文
- **eas-entry** - 应用入口模块

---

## 2. DDD战略设计

### 2.1 限界上下文（Bounded Context）
项目通过Maven模块清晰划分限界上下文：
- **培训上下文** - 管理培训课程、培训票、提名流程
- **员工上下文** - 管理员工信息
- **项目上下文** - 管理项目信息

### 2.2 子域（Sub Domain）
- **核心域** - 培训管理
- **支撑域** - 员工管理、项目管理

---

## 3. DDD战术设计

### 3.1 分层架构
项目采用典型的DDD分层架构：

```
┌─────────────────────────────────────┐
│    用户界面层 (UI/REST Resources)    │
├─────────────────────────────────────┤
│      应用层 (Application Service)     │
├─────────────────────────────────────┤
│        领域层 (Domain Layer)         │
│  聚合/实体/值对象/领域服务/领域事件  │
├─────────────────────────────────────┤
│    基础设施层 (Infrastructure Layer)  │
└─────────────────────────────────────┘
```

### 3.2 聚合（Aggregate）
项目使用`@Aggregate`注解标识聚合根：

#### 培训聚合 ([Training](https://github.com/huoxudong125/eas-ddd/blob/master/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/training/Training.java))
```java
@Aggregate
public class Training {
    private TrainingId id;
    private String title;
    private String description;
    private LocalDateTime beginTime;
    private LocalDateTime endTime;
    private String place;
    private CourseId courseId;
    // ...
}
```

#### 培训票聚合 ([Ticket](https://github.com/huoxudong125/eas-ddd/blob/master/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/ticket/Ticket.java))
```java
@Aggregate
public class Ticket {
    private TicketId id;
    private TrainingId trainingId;
    private TicketStatus ticketStatus;
    private String nomineeId;
    
    public TicketHistory nominate(Candidate candidate, Nominator nominator) {
        validateTicketStatus();
        doNomination(candidate);
        return generateHistory(candidate, nominator);
    }
    // ...
}
```

**聚合特点：**
- 通过聚合根（Aggregate Root）统一访问
- 内部保持一致性边界
- 业务逻辑封装在聚合内部

### 3.3 值对象（Value Object）
#### TicketId 值对象 ([TicketId](https://github.com/huoxudong125/eas-ddd/blob/master/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/ticket/TicketId.java))
```java
public class TicketId {
    private String value;
    
    private TicketId(String value) {
        this.value = value;
    }
    
    public static TicketId next() {
        return new TicketId(UUID.randomUUID().toString());
    }
    
    public static TicketId from(String value) {
        return new TicketId(value);
    }
    
    public String value() {
        return value;
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        TicketId ticketId = (TicketId) o;
        return value.equals(ticketId.value);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(value);
    }
}
```

**值对象特点：**
- 不可变性（Immutable）
- 通过值相等性比较
- 封装值验证逻辑
- 工厂方法创建

#### 身份标识基类 ([AbstractIdentity](https://github.com/huoxudong125/eas-ddd/blob/master/ddd-core/src/main/java/xyz/zhangyi/ddd/core/domain/AbstractIdentity.java))
```java
public abstract class AbstractIdentity<T> implements Identity<T> {
    private T value;
    
    protected AbstractIdentity(T value) {
        this.setId(value);
    }
    
    private void setId(T value) {
        if (value == null) {
            throw new IllegalArgumentException("The identity is required ");
        }
        this.validateValue(value);
        this.value = value;
    }
    
    protected void validateValue(T value) {
        // 子类可覆盖实现验证逻辑
    }
    
    // equals, hashCode, toString...
}
```

### 3.4 领域服务（Domain Service）
项目使用`@DomainService`注解标识领域服务：

#### 提名服务 ([NominationService](https://github.com/huoxudong125/eas-ddd/blob/master/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/ticket/NominationService.java))
```java
@Service
@DomainService
public class NominationService {
    @Autowired
    private LearningService learningService;
    @Autowired
    private TicketService ticketService;
    @Autowired
    private NotificationService notificationService;
    
    public void nominate(TicketId ticketId, TrainingId trainingId, 
                        Candidate candidate, Nominator nominator) {
        boolean beLearned = learningService.beLearned(candidate.employeeId(), trainingId);
        if (beLearned) {
            throw new NominationException(...);
        }
        Ticket ticket = ticketService.nominate(ticketId, nominator, candidate);
        notificationService.notifyNominee(ticket, nominator, candidate.toNominee());
    }
}
```

**领域服务特点：**
- 处理跨聚合的业务逻辑
- 无状态
- 业务语义清晰

### 3.5 仓储（Repository）
项目采用**端口-适配器模式**实现仓储：

#### 仓储端口 ([TicketRepository](https://github.com/huoxudong125/eas-ddd/blob/master/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/south/port/repository/TicketRepository.java))
```java
@Mapper
@Repository
@Port(PortType.Repository)
public interface TicketRepository {
    Optional<Ticket> ticketOf(TicketId ticketId, TicketStatus ticketStatus);
    void update(Ticket ticket);
    void add(Ticket ticket);
    void remove(Ticket ticket);
}
```

**仓储特点：**
- 接口定义在领域层
- 实现在基础设施层
- 使用`@Port`注解标识端口类型
- 面向聚合操作

---

## 4. 六边形架构（端口-适配器）

项目采用六边形架构实现解耦：

### 4.1 北向网关（North Gateway）
对外暴露的接口：
- **REST资源** - [TicketResource](https://github.com/huoxudong125/eas-ddd/blob/master/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/north/remote/resource/TicketResource.java)
- **应用服务** - [NominationAppService](https://github.com/huoxudong125/eas-ddd/blob/master/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/north/local/appservice/NominationAppService.java)

#### 应用服务示例
```java
@Service
@EnableTransactionManagement
@Local
public class NominationAppService {
    @Autowired
    private NominationService nominationService;
    
    @Transactional(rollbackFor = ApplicationException.class)
    public void nominate(NominatingCandidateRequest nominatingCandidateRequest) {
        if (Objects.isNull(nominatingCandidateRequest)) {
            throw new ApplicationValidationException("nomination request can not be null");
        }
        try {
            nominationService.nominate(...);
        } catch (DomainException ex) {
            throw new ApplicationDomainException(ex.getMessage(), ex);
        } catch (Exception ex) {
            throw new ApplicationInfrastructureException("Infrastructure Error", ex);
        }
    }
}
```

### 4.2 南向网关（South Gateway）
与外部系统交互的接口：
- **仓储端口** - 数据持久化
- **外部客户端端口** - [NotificationClient](https://github.com/huoxudong125/eas-ddd/blob/master/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/south/port/client/NotificationClient.java)

### 4.3 注解体系
项目定义了丰富的注解体系：

| 注解 | 用途 |
|------|------|
| [@Aggregate](https://github.com/huoxudong125/eas-ddd/blob/master/ddd-core/src/main/java/xyz/zhangyi/ddd/core/stereotype/Aggregate.java) | 标识聚合根 |
| [@DomainService](https://github.com/huoxudong125/eas-ddd/blob/master/ddd-core/src/main/java/xyz/zhangyi/ddd/core/stereotype/DomainService.java) | 标识领域服务 |
| [@Port](https://github.com/huoxudong125/eas-ddd/blob/master/ddd-core/src/main/java/xyz/zhangyi/ddd/core/stereotype/Port.java) | 标识端口 |
| [@Adapter](https://github.com/huoxudong125/eas-ddd/blob/master/ddd-core/src/main/java/xyz/zhangyi/ddd/core/stereotype/Adapter.java) | 标识适配器 |
| [@Remote](https://github.com/huoxudong125/eas-ddd/blob/master/ddd-core/src/main/java/xyz/zhangyi/ddd/core/stereotype/Remote.java) | 标识远程资源 |
| [@Local](https://github.com/huoxudong125/eas-ddd/blob/master/ddd-core/src/main/java/xyz/zhangyi/ddd/core/stereotype/Local.java) | 标识本地服务 |

---

## 5. 异常体系

项目定义了完整的异常层级结构：

```
ApplicationException (应用异常基类)
├── ApplicationDomainException (领域异常)
├── ApplicationInfrastructureException (基础设施异常)
└── ApplicationValidationException (验证异常)

DomainException (领域异常基类)
```

---

## 6. 测试策略

### 6.1 测试分类
- **单元测试** - `*Test.java` - 测试领域模型
- **集成测试** - `*IT.java` - 测试应用服务和资源层

### 6.2 领域模型测试示例
[TicketTest](https://github.com/huoxudong125/eas-ddd/blob/master/eas-training/src/test/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/ticket/TicketTest.java)
```java
public class TicketTest {
    @Test
    public void should_generate_ticket_history_after_ticket_was_nominated() {
        Ticket ticket = new Ticket(TicketId.next(), trainingId);
        TicketHistory ticketHistory = ticket.nominate(candidate, nominator);
        assertTicketHistory(ticket, ticketHistory);
    }
    
    @Test
    public void should_throw_TicketException_given_ticket_is_not_AVAILABLE() {
        Ticket ticket = new Ticket(TicketId.next(), trainingId, TicketStatus.WaitForConfirm);
        
        assertThatThrownBy(() -> ticket.nominate(candidate, nominator))
                .isInstanceOf(TicketException.class)
                .hasMessageContaining("ticket is not available");
    }
}
```

---

## 7. 核心设计模式

| 模式 | 应用 |
|------|------|
| 值对象 | TicketId, TrainingId |
| 聚合 | Training, Ticket |
| 领域服务 | NominationService, TicketService |
| 仓储 | TicketRepository, TrainingRepository |
| 端口-适配器 | NotificationClient及其实现 |
| 工厂方法 | TicketId.next(), TicketId.from() |

---

## 8. 关键文件速查

### 核心框架
- [AbstractIdentity.java](https://github.com/huoxudong125/eas-ddd/blob/master/ddd-core/src/main/java/xyz/zhangyi/ddd/core/domain/AbstractIdentity.java) - ID基类
- [Aggregate.java](https://github.com/huoxudong125/eas-ddd/blob/master/ddd-core/src/main/java/xyz/zhangyi/ddd/core/stereotype/Aggregate.java) - 聚合注解
- [Port.java](https://github.com/huoxudong125/eas-ddd/blob/master/ddd-core/src/main/java/xyz/zhangyi/ddd/core/stereotype/Port.java) - 端口注解

### 培训上下文
- [Ticket.java](https://github.com/huoxudong125/eas-ddd/blob/master/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/ticket/Ticket.java) - 培训票聚合
- [Training.java](https://github.com/huoxudong125/eas-ddd/blob/master/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/training/Training.java) - 培训聚合
- [NominationService.java](https://github.com/huoxudong125/eas-ddd/blob/master/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/domain/ticket/NominationService.java) - 提名服务
- [NominationAppService.java](https://github.com/huoxudong125/eas-ddd/blob/master/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/north/local/appservice/NominationAppService.java) - 提名应用服务
- [TicketResource.java](https://github.com/huoxudong125/eas-ddd/blob/master/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/north/remote/resource/TicketResource.java) - 票资源
- [TicketRepository.java](https://github.com/huoxudong125/eas-ddd/blob/master/eas-training/src/main/java/xyz/zhangyi/ddd/eas/valueadded/trainingcontext/south/port/repository/TicketRepository.java) - 票仓储

---

## 9. 学习路径建议

1. **理解战略设计**：先看模块划分和限界上下文
2. **学习战术设计**：从ddd-core模块开始，了解核心抽象
3. **深入业务实现**：以eas-training模块为核心，理解聚合、值对象、领域服务
4. **理解架构模式**：分析端口-适配器模式的实现
5. **学习测试策略**：查看测试代码，理解如何测试DDD模型
6. **动手实践**：尝试运行项目，添加新功能

---

## 10. 参考资源

- [项目Wiki](https://github.com/huoxudong125/eas-ddd/wiki)
- GitChat课程：《领域驱动设计实战（战略篇）》《领域驱动设计实战（战术篇）》
- [JPA版本参考项目](https://github.com/huoxudong125/payroll-ddd)

---

这份文档涵盖了EAS-DDD项目的核心DDD设计理念和实现细节。通过学习这个项目，你可以深入理解如何在实际项目中应用DDD的战略设计和战术设计模式。