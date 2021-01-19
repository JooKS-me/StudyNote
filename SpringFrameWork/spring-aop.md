#### AOP（面向切面编程）基本概念

- 一个概念/规范，没有限定语言
- 不是取代OOP编程，而是OOP的补充，和数据库的触发器有点相似
- Aspect ：配置文件，包括一些Pointcut和相应的Advice
- Joint point：在程序中明确定义的点，如方法调用、对类成员访问等
- Pointcut：一组joint point, 可以通过逻辑关系/通配符/正则等组合起来，定义 了相应advice将要发生的地方
- Advice：定义了在pointcut处要发生的动作,通过before/after/around/来关联
- weaving：advice代码在具体joint point的关联方式（如动态代理、字节码等）

---

首先，需要导入两个依赖。

```xml
		<dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.5</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjrt</artifactId>
            <version>1.9.5</version>
        </dependency>
```

## 法一：使用原生Spring API接口

个人感觉这种方式比较散乱，没有核心。

首先，实现接口

```java
public class AfterLog implements AfterReturningAdvice {
    public void afterReturning(Object o, Method method, Object[] objects, Object o1) throws Throwable {
        System.out.println("执行了" + method.getName() + "方法，返回值为" + o);
    }
}
```

```java
public class Log implements MethodBeforeAdvice {
    public void before(Method method, Object[] objects, Object o) throws Throwable {
        System.out.println(o.getClass().getName() + "的" + method.getName());
    }
}
```

然后，注册好Bean

（略）

最后，写好xml配置（记得头部依赖需要写好）

```xml
	<aop:config>
        <aop:pointcut id="pointcut" expression="execution(* com.jooks.service.UserServiceImpl.*(..))"/>

        <aop:advisor advice-ref="log" pointcut-ref="pointcut"/>
        <aop:advisor advice-ref="afterLog" pointcut-ref="pointcut"/>
    </aop:config>
```

## 法二：以切面(Aspect)为核心，基于xml配置

首先，写好Aspect类（只包含Aspect的所有advice）

```java
public class DiyAdvice {
    public void before() {
        System.out.println("==========前置操作==========");
    }

    public void after() {
        System.out.println("==========后置操作===========");
    }
}
```

然后，注册Bean

（略）

然后，写好xml配置即可

```xml
    <aop:config>
        <aop:aspect ref="DiyAdvice">
            <aop:pointcut id="point" expression="execution(* com.jooks.service.UserService.*(..))"/>
            <aop:before method="before" pointcut-ref="point"/>
            <aop:after method="after" pointcut-ref="point"/>
        </aop:aspect>
    </aop:config>
```

## 法三：以切面(Aspect)为核心，基于注解

首先，在xml配置文件中写好自动代理标签

```xml
<aop:aspectj-autoproxy/>
```

然后，编写Aspect类

```java
@Aspect
public class AnnocationPointcut {
    @Before("execution(* com.jooks.service.UserService.*(..))")
    public void before(JoinPoint joinPoint) {
        System.out.println("============执行前===========");
        System.out.println("目标方法：" + joinPoint.getSignature().getDeclaringTypeName() + "." + joinPoint.getSignature().getName());
        System.out.println("参数：" + Arrays.toString(joinPoint.getArgs()));
        System.out.println("目标对象：" + joinPoint.getTarget());
        System.out.println("============================");
    }

    @After("execution(* com.jooks.service.UserService.*(..))")
    public void after() {
        System.out.println("============执行后===========");
    }

    @Around("execution(* com.jooks.service.UserService.*(..))")
    public void round(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("环绕前");
        Object proceed = joinPoint.proceed();
        System.out.println("环绕后");
    }
}
```

最后，注册Bean即可

## AOP织入事务（声明式事务）

直接配置xml即可

```xml
	<bean id="transActionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>    
	<tx:advice id="txAdvice" transaction-manager="transActionManager">
        <tx:attributes>
<!--            <tx:method name="queryAll" read-only="true"/>-->
<!--            <tx:method name="add" propagation="REQUIRED"/>-->
<!--            <tx:method name="delete" propagation="REQUIRED"/>-->
            <tx:method name="*" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>

    <aop:config>
        <aop:pointcut id="txPointCut" expression="execution(* com.jooks.dao.*.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointCut"/>
    </aop:config>
```
