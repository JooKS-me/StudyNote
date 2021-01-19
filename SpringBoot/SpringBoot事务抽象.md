> spring的事务抽象：不管你是使用JDBC/Hibernate/MyBatis来操作数据，还是使用DataSource/JTA的事务，都能通过事务抽象统一在一起。

## 事务抽象的核心接口

**PlatformTransactionManager**

- DataSourceTransactionManager（extend AbstractPlatformTransactionManager => implements PlatformTransactionManager)
- HibernateTransactionManager
- ...

PlatformTransactionManager源码如下

```java
// TransactionManager是一个marker interface，里什么都没有
public interface PlatformTransactionManager extends TransactionManager {
	TransactionStatus getTransaction(@Nullable TransactionDefinition definition)
			throws TransactionException;
	
	void commit(TransactionStatus status) throws TransactionException;
	
	void rollback(TransactionStatus status) throws TransactionException;
}
```

**其中的TransactionDefinition**

- Propagation，传播特性

  spring有7个传播特性：

  - PROPAGATION_REQUIRED (**required**)，0，当前有事务就用当前的，没有就用新的。（默认）
  - PROPAGATION_SUPPORTS (supports)，1，事务可有可无，不是必须的。
  - PROPAGATION_MANDATORY (mandatory)，2，一定要有事务，不然就抛异常。
  - PROPAGATION_REQUIRES_NEW (**requires new**)，3，无论是否有事务，都起个新的事务。
  - PROPAGATION_NOT_SUPPORTED (not supproted)，4，不支持事务，按非事务方式运行。
  - PROPAGATION_NEVER (never)，5，不支持事务，如果有事务则抛异常。
  - PROPAGATION_NESTED(**nested**)，6，当前有事务就在当前事务里再起一个事务。

- Isolation，隔离性

  spring有4个事务隔离特性：

  |                隔离性                 |  值  | 脏读 | 不可重复读 | 幻读 |
  | :-----------------------------------: | :--: | :--: | :--------: | :--: |
  |      ISOLATION_READ_UNCOMMITTED       |  1   |  √   |     √      |  √   |
  |       ISOLATION_READ_COMMITTED        |  2   |  ×   |     √      |  √   |
  | ISOLATION_REPEATABLE_READ(repeatable) |  3   |  ×   |     ×      |  √   |
  | ISOLATION_SERIALIZABLE(serializable)  |  4   |  ×   |     ×      |  ×   |

- Timeout，超时设置
- Read-only status，是否只读

## 编程式事务

**TransactionTemplate**

- TransactionCallback，有返回值
- TransactionCallbackWithoutResult，无返回值

**PlatformTransactionManager**

- 可以传入TransactionDefinition进行定义

TransactionTemplate中的一个构造函数如下

```java
public TransactionTemplate(PlatformTransactionManager transactionManager, TransactionDefinition transactionDefinition) {
   super(transactionDefinition);
   this.transactionManager = transactionManager;
}
```

Demo如下

```java
@SpringBootApplication
@Slf4j
public class Springboot01TransactionApplication implements CommandLineRunner {
   @Autowired
   private TransactionTemplate transactionTemplate;
   @Autowired
   private JdbcTemplate jdbcTemplate;

   public static void main(String[] args) {
      SpringApplication.run(Springboot01TransactionApplication.class, args);
   }

   @Override
   public void run(String... args) throws Exception {
      log.info("count before transaction: {}", getCount());
      transactionTemplate.execute(new TransactionCallbackWithoutResult() {
         @Override
         protected void doInTransactionWithoutResult(TransactionStatus status) {
            jdbcTemplate.execute("INSERT INTO FOO (ID, BAR) VALUES (1, 'AAA')");
            log.info("count in transaction: {}", getCount());
            status.setRollbackOnly();
         }
      });
      log.info("count after transaction: {}", getCount());
   }

   private long getCount() {
      return (long) jdbcTemplate.queryForList("SELECT COUNT(*) AS CNT FROM FOO").get(0).get("CNT");
   }
}
```

## 声明式事务

下面都是使用注解的方式

**开启事务注解的方式（二选一）**

- 注解：@EnableTransactionManagement
- xml配置：`<tx:annotation-driven/>`

**一些配置**

- proxyTargetClass，有接口时使用。
  - 若没有接口，直接是实现类时，可以使用CGLib直接对类做增强。
- mode，AspectJ或Java的AOP
- order，指定事务AOP拦截的顺序，默认是最低的，使得自定义的AOP拦截可以在事务开启后执行

**@Transactional**：添加在需要的方法或类

- transactionManger
- propagation
- isolation
- timeout
- readOnly
- rollbackFor，指定什么异常回滚

Demo如下

```java
@Service
public class FooServiceImpl implements FooService {
    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Override
    @Transactional
    public void insertRecord() {
        jdbcTemplate.execute("INSERT INTO FOO (BAR) VALUES ('AAA')");
    }

    @Override
    @Transactional(rollbackFor = RollbackException.class)
    public void insertThenRollback() throws RollbackException {
        jdbcTemplate.execute("INSERT INTO FOO (BAR) VALUES ('BBB')");
        throw new RollbackException();
    }

    @Override
    public void invokeInsertThenRollback() throws RollbackException {
        insertThenRollback();
    }
}
```

```java
@SpringBootApplication
@Slf4j
@EnableTransactionManagement
public class Springboot01TransactionApplication implements CommandLineRunner {
   @Autowired
   private JdbcTemplate jdbcTemplate;
   @Autowired
   private FooService fooService;

   public static void main(String[] args) {
      SpringApplication.run(Springboot01TransactionApplication.class, args);
   }

   @Override
   public void run(String... args) throws Exception {
      fooService.insertRecord();
      log.info("AAA {}",
            jdbcTemplate.queryForObject("SELECT COUNT(*) FROM FOO WHERE BAR = 'AAA'", Long.class));

      try {
         fooService.insertThenRollback();
      } catch (RollbackException e) {
         e.printStackTrace();
         log.info("BBB {}",
               jdbcTemplate.queryForObject("SELECT COUNT(*) FROM FOO WHERE BAR='BBB'", Long.class));
      }

      try {
         fooService.invokeInsertThenRollback();
      } catch (RollbackException e) {
         e.printStackTrace();
         log.info("BBB {}",
               jdbcTemplate.queryForObject("SELECT COUNT(*) FROM FOO WHERE BAR='BBB'", Long.class));
      }
   }
}
```

上面的demo会在日志中打印：

AAA 1

BBB 0

BBB 1



为什么最后一个会输出BBB 1呢，也就是为什么invokeInsertThenRollback()没有开启事务呢？

答曰：Spring为你的类做了一个代理，也就是说你需要去调用代理类才能真正的执行被代理的增强的那些方法。如果该方法的调用是在类的内部调用的话，也就意味着该方法的调用不是通过代理类的调用。而Spring的声明式事务本质上是通过AOP来增强了类的功能。



关于多线程事务问题，老师能讲一下吗？

答曰：多线程之间最好事务隔离开，线程里用自己的事务。