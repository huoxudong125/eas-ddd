[![Build Status](https://travis-ci.org/agiledon/eas-ddd.svg?branch=master)](https://travis-ci.org/agiledon/eas-ddd)
[![codecov](https://codecov.io/gh/agiledon/eas-ddd/branch/master/graph/badge.svg)](https://codecov.io/gh/agiledon/eas-ddd)

### Project Introduction

EAS-DDD is a practical project case provided for my GitChat courses "[Domain-Driven Design in Practice (Strategic Volume)](https://gitbook.cn/gitchat/column/5b3235082ab5224deb750e02)" and "[Domain-Driven Design in Practice (Tactical Volume)](https://gitbook.cn/gitchat/column/5cbed2f6f00736695f3a8699)". This case is based on a real project I participated in, and the entire code of the case is modeled, designed, and coded completely according to the domain-driven design process.

By visiting the [Wiki](https://github.com/agiledon/eas-ddd/wiki) corresponding to this Repository, you can understand the requirements, modeling process, and modeling deliverables of the EAS project; for more detailed analysis and design content, please subscribe to my courses published on GitChat. For user stories and split tasks corresponding to each domain scenario, please visit the [Issue](https://github.com/agiledon/eas-ddd/issues) of this Repository.

### Environment

This project is developed based on the Java language, and the specific environment includes:

```
Java: Java 8+
Maven: 3
Spring: 5.1.10+
Spring Boot：2.1.9
MyBatis：3.5.3
Druid：1.1.20
MySQL: 8.0 Community
```

I personally believe that JPA ORM is more in line with DDD design. In another project [Payroll-DDD](https://github.com/agiledon/payroll-ddd) developed with DDD, the persistence framework I chose is Spring Data JPA. The reason why this project uses MyBatis is that MyBatis is more common in the field of enterprise software development in China. At the same time, I also hope to show through this project that MyBatis can also be used as a persistence framework for DDD.

### Database Environment Preparation

The default database username for the project is sa, the password is 123456, the database host is localhost, and the database is eas-db. After installing MySQL 8.0, if the database server information is different from the default information, please modify the following files. When using flywaydb to execute database scripts, you need to ensure that the database configuration is correct and the eas-db database has been created.

**Database configuration for flywaydb**

The following content is configured in `<plugins>` in the `pom.xml` file:

```xml
<plugin>
   <groupId>org.flywaydb</groupId>
   <artifactId>flyway-maven-plugin</artifactId>
   <version>${flyway.version}</version>
   <configuration>
        <driver>${db.driver}</driver>
        <url>${db.url}</url>
        <user>${db.username}</user>
        <password>${db.password}</password>
   </configuration>
</plugin>
```

In the properties configuration part of the same pom file, the relevant properties of the database are configured:

```xml
<properties>
    <db.driver>com.mysql.jdbc.Driver</db.driver>
    <db.url>jdbc:mysql://localhost:3306/eas-db?serverTimezone=UTC</db.url>
    <db.username>sa</db.username>
    <db.password>123456</db.password>
</properties>
```

Once the flywaydb environment is ready, you can run commands to perform DB cleanup:

```
mvn package flyway:clean
```

Or execute the command to perform DB migration:

```
mvn package flyway:migrate
```

To ignore running unit tests, you can add parameters after the Maven command:

```
-DskipTests
```

**Test Environment Preparation**

This project uses test-driven development to implement the classes and methods of the domain layer, so the relevant methods of the domain layer are basically covered by unit tests.

Application services ensure their correctness through integration testing. To run integration tests, you need to execute SQL scripts through flywaydb to prepare the database environment necessary for integration testing. If the database configuration is different from my configuration, in addition to modifying the flywaydb configuration in the `pom.xml` file, you also need to configure the relevant files in the `test/resources/spring-mybatis.xml` file of each bounded context module, such as:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">
    <context:component-scan base-package="xyz.zhangyi.ddd.eas.trainingcontext" />

    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
          init-method="init" destroy-method="close">
        <property name="url" value="jdbc:mysql://localhost:3306/eas-db?serverTimezone=UTC"/>
        <property name="username" value="sa"/>
        <property name="password" value="123456"/>
        <property name="connectionProperties" value="com.mysql.jdbc.Driver"/>
    </bean>
</beans>    
```

#### Running Tests

By default, running `mvn test` will only run unit tests. If you ensure that the database is ready and that the database table structure and test data are ready through flywaydb, you can run `mvn integration-test`. This command will run all tests, including unit tests and integration tests.

**Note:** All unit tests in the project use `Test` as the test class suffix, and all integration tests use `IT` as the test class suffix.

### Running Spring Boot Service

The entire project adopts a monolithic architecture, so all remote services of bounded contexts pass through a unified main program entrance, that is, EasApplication under the eas-entry module. The application configuration of Spring Boot is in the resources folder of this module, the file name is `application.yml`, and the content is:

```yml
mybatis:
  mapperLocations: classpath:mapper/*.xml
  type-aliases-package: xyz.zhangyi.ddd.eas.trainingcontext.domain
  type-handlers-package: xyz.zhangyi.ddd.eas.trainingcontext.gateway.acl.impl.persistence.typehandlers

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/eas-db?serverTimezone=UTC
    username: sa
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
    # Use druid data source
    type: com.alibaba.druid.pool.DruidDataSource
```

To start the Spring Boot service, you need to ensure that the database configuration is correct. If the port is not configured, the default port number is 8080. You can also specify the port in `application.yml`.

You can directly run `EasApplication` in the IDE to start the Spring Boot service.

As you can see, the current project adopts a monolithic architecture, but it can be easily migrated to a microservice architecture. If you use Spring Boot to publish services, you need to add the dependency of `spring-boot-starter` to the `pom.xml` file of each module. You can refer to the dependency configuration of the `eas-entry` module.
