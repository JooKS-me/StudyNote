> https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter

## 一、加入starter依赖

**注意一定要在spring-boot-starter-jdbc中排除默认的HikariCP连接池**

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-jdbc</artifactId>
			<exclusions>
				<exclusion>
					<artifactId>HikariCP</artifactId>
					<groupId>com.zaxxer</groupId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid-spring-boot-starter</artifactId>
			<version>1.1.22</version>
		</dependency>
```

## 二、配置好application.properties

具体看Druid官方文档：

https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter

```properties
spring.output.ansi.enabled=ALWAYS

spring.datasource.url=jdbc:h2:mem:foo
spring.datasource.username=sa
spring.datasource.password=n/z7PyA5cvcXvs8px8FVmBVpaRyNsvJb3X7YfS38DJrIg25EbZaZGvH4aHcnc97Om0islpCAPc3MqsGvsrxVJw==

spring.datasource.druid.initial-size=5
spring.datasource.druid.max-active=5
spring.datasource.druid.min-idle=5
spring.datasource.druid.filters=conn,config,stat,slf4j

spring.datasource.druid.connection-properties=config.decrypt=true;config.decrypt.key=${public-key}
spring.datasource.druid.filter.config.enabled=true

spring.datasource.druid.test-on-borrow=true
spring.datasource.druid.test-on-return=true
spring.datasource.druid.test-while-idle=true

public-key=MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBALS8ng1XvgHrdOgm4pxrnUdt3sXtu/E8My9KzX8sXlz+mXRZQCop7NVQLne25pXHtZoDYuMh3bzoGj6v5HvvAQ8CAwEAAQ==
```

## 三、使用Druid Filter

用于定制连接池的各种操作

1. 通过继承FilterEventAdapter以方便地实现Filter

   ```java
   @Slf4j
   public class ConnectionLogFilter extends FilterEventAdapter {
   
       /* 在连接前后打印日志 */
       
       @Override
       public void connection_connectBefore(FilterChain chain, Properties info) {
           log.info("BEFORE CONNECTION!");
       }
   
       @Override
       public void connection_connectAfter(ConnectionProxy connection) {
           log.info("AFTER CONNECTION!");
       }
   }
   ```

2. 修改META-INF/druid-filter.properties增加Filter配置

   ```properties
   druid.filters.conn=geektime.spring.data.druiddemo.ConnectionLogFilter
   ```

## 慢SQL日志

Druid的监控功能可以方便我们找出慢sql

可以通过**系统属性配置**，在java启动时加上-d，加上下面的东西：

- druid.stat.logSlowSql=true
- druid.stat.slowSqlMillis=3000

或者在Spring Boot的配置文件中加上：

- spring.datasource.druid.filter.stat.enabled=true（可以不加，默认是true）
- spring.datasource.druid.filter.stat.log-slow-sql=true
- spring.datasource.druid.filter.stat.slow-sql-millis=3000

## 一些注意事项

- 没特殊情况，不要在生产环境打开监控的Servlet
- 没有连接泄露的情况下，不要开启removeAbandoned，因为它会对你的性能造成极大影响
- testXxx 使用时需注意，开销挺大的
- 务必配置合理的超时时间



---

想问下老师，JDBC,hibernate，jpa，druid，sparding-shpere，HikariCP这些东西到底有啥区别啊，平时是不是只用其中一个就够了？感觉要蒙了

答曰：显然你真的是没有搞明白这些东西的含义……JPA和JDBC都是规范，JPA可以看做ORM的一套规范，Hibernate是JPA的实现；JDBC提供了Java数据库操作的底层规范，各种连接啊、查询啊什么的操作都是JDBC来定义的，不同数据库都提供了遵循JDBC的驱动；Druid和HikariCP都是数据源，或者简单点说是数据库连接池；ShardingShpere是用来做分库分表。平时这些东西是要根据实际情况结合在一起来使用。

