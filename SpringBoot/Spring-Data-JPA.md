JPA：Java Persistence API

JPA为对象关系映射提供了一种基于POJO的持久化模型

## 常见JPA注解

**实体**

- @Entity，注明实体，如果不加name=xxx或table注解，那么类目就是表名
- @MaapedSuperclass，标注父类
- @Table(name)，把实体与表对应起来

**主键**

- @Id，指定主键
  - @GeneratedValue(strategy, generator)，指定自动生成的主键
  - @SequenceGenerator(name, sequenceName)，指定序列生成的主键

**映射**

- @Column(name, nullable, length, insertable, updatable)
- @JoinTable(name)、@JoinColumn(name)

**关系**

- @OneToOne、@OneToMany、@ManyToOne、@ManyToMany
- @OrderBy

## 实体定义

也就是一些注解+可以继承

Demo如下

```java
@MappedSuperclass
@Data
@NoArgsConstructor
@AllArgsConstructor
public class BaseEntity implements Serializable {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY) //自增策略
    private Long id;

    @Column(updatable = false)
    @CreationTimestamp
    private Date createTime;

    @UpdateTimestamp
    private Date updateTime;
}
```

```java
@Entity
@Table(name = "T_MENU")
@Builder
@Data
@ToString(callSuper = true) //默认不会加上父类
@NoArgsConstructor
@AllArgsConstructor
public class Coffee extends BaseEntity implements Serializable {
    private String name;

    @Column
    @Type(type = "org.jadira.usertype.moneyandcurrency.joda.PersistentMoneyMinorAmount",
            parameters = {@org.hibernate.annotations.Parameter(name = "currencyCode", value = "CNY")})
    private Money price;
}
```

```java
@Entity
@Table(name = "T_ORDER")
@Data
@ToString(callSuper = true)
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class CoffeeOrder extends BaseEntity implements Serializable {
    private String customer;

    @ManyToMany
    @JoinTable(name = "T_ORDER_COFFEE")
    @OrderBy("id")
    private List<Coffee> items;

    @Enumerated //表示使用枚举类
    @Column(nullable = false)
    private OrderState state;
}
```

```java
public enum OrderState {
    INIT, PAID, BREWING, BREWED, TAKEN, CANCELLED
}
```

运行后，Hibernate会帮我们自动生成并执行SQL

## CRUD

**@EnableJpaRepositories**：开启jpa的crud功能

直接使用**Repository<T, ID>**接口：创建接口继承即可

- CrudRepository<T, ID>
- PagingAndSortingRepository<T, ID>，继承自CrudRepository
- JpaRepository<T, ID>

或者**自定义查询**

**根据方法名定义查询**

- find...By... / read...By... / query...By... / get...By... 简单查找
- count...By... 计数
- ...OrderBy...[Asc / Desc] 定义排序
- And / Or / IgnoreCase
- Top / First / Distinct 

**分页查询**

- PagingAndSortingRepository<T, ID>接口
- Pageable / Sort 类
- Slice< T > / Page< T >

插入Demo如下

```java
public interface CoffeeRepository extends CrudRepository<Coffee, Long> {
}
```

```java
public interface CoffeeOrderRepository extends CrudRepository<CoffeeOrder, Long> {
}
```

```java
@EnableJpaRepositories
@SpringBootApplication
public class SpringbootStudy02JpaApplication implements ApplicationRunner {
   @Autowired
   private CoffeeRepository coffeeRepository;
   @Autowired
   private CoffeeOrderRepository coffeeOrderRepository;

   public static void main(String[] args) {
      SpringApplication.run(SpringbootStudy02JpaApplication.class, args);
   }

   @Override
   public void run(ApplicationArguments args) throws Exception {
      initOrders();
   }

   private void initOrders() {
      Coffee coffee1 = Coffee.builder().name("coffee1").price(Money.of(CurrencyUnit.of("CNY"), 20.0)).build();
      coffeeRepository.save(coffee1);

      Coffee coffee2 = Coffee.builder().name("coffee2").price(Money.of(CurrencyUnit.of("CNY"), 30.0)).build();
      coffeeRepository.save(coffee2);

      CoffeeOrder coffeeOrder1 = CoffeeOrder.builder()
                                 .customer("Linda")
                                 .items(Arrays.asList(coffee1, coffee2))
                                 .state(OrderState.INIT)
                                 .build();
      coffeeOrderRepository.save(coffeeOrder1);
   }
}
```

查询Demo如下

```java
@NoRepositoryBean
public interface BaseRepository<T, Long> extends PagingAndSortingRepository<T, Long> {
    List<T> findTop3ByOrderByUpdateTimeDescIdAsc();
}
```

```java
public interface CoffeeOrderRepository extends BaseRepository<CoffeeOrder, Long> {
    List<CoffeeOrder> findByCustomerOrderById(String customer);
    List<CoffeeOrder> findByItems_Name(String name);
}
```

```java
public interface CoffeeRepository extends BaseRepository<Coffee, Long> {
}
```

```java
@SpringBootApplication
@EnableJpaRepositories
@EnableTransactionManagement
@Slf4j
public class JpaDemoApplication implements ApplicationRunner {
	@Autowired
	private CoffeeRepository coffeeRepository;
	@Autowired
	private CoffeeOrderRepository orderRepository;

	public static void main(String[] args) {
		SpringApplication.run(JpaDemoApplication.class, args);
	}

	@Override
	@Transactional
	public void run(ApplicationArguments args) throws Exception {
		initOrders();
		findOrders();
	}

	private void initOrders() {
		// 同Demo1
	}

	private void findOrders() {
		coffeeRepository
				.findAll(Sort.by(Sort.Direction.DESC, "id"))
				.forEach(c -> log.info("Loading {}", c));

		List<CoffeeOrder> list = orderRepository.findTop3ByOrderByUpdateTimeDescIdAsc();
		log.info("findTop3ByOrderByUpdateTimeDescIdAsc: {}", getJoinedOrderId(list));

		list = orderRepository.findByCustomerOrderById("Li Lei");
		log.info("findByCustomerOrderById: {}", getJoinedOrderId(list));

		// 不开启事务会因为没Session而报LazyInitializationException
		list.forEach(o -> {
			log.info("Order {}", o.getId());
			o.getItems().forEach(i -> log.info("  Item {}", i));
		});

		list = orderRepository.findByItems_Name("latte");
		log.info("findByItems_Name: {}", getJoinedOrderId(list));
	}

	private String getJoinedOrderId(List<CoffeeOrder> list) {
		return list.stream().map(o -> o.getId().toString())
				.collect(Collectors.joining(","));
	}
}
```



---

老师可以解释一下那个懒加载的问题吗？

答曰：这里的集合Hibernate给我们做了个LazyLoading的优化，在使用时才会去加载，没有事务时findByCustomerOrderById操作后连接就还回去了，Session也关了，后面访问items时自然就没有Session报错了。加了事务，连接在我提交或者回滚前都会持有，Session也在，自然就不会报错了。也可以设置@ManyToMany的fetch，默认是LAZY。

## Repository是如何被创建的

**JpaRepositoriesRegistrar**

- 激活了@EnableJpaRepositories
- 返回了JpaRepositoryConfigExtension

**RepositoryBeanDefinitionRegistrarSupport.registerBeanDefinitions**

- 注册Repository Bean（类型是JpaRepositoryFactoryBean)

**RepositoryConfigurationExtensionSupport.getRepositoryConfigurations**

- 取得Repository配置

**JpaRepositoryFactory.getTargetRepository**

- 创建了Repository

## 接口中的方法是如何被解释的

**RepositoryFactorySupport.getRepository 添加了 Advice**

- DefaultMethodInvokingMethodInterceptor
- QueryExecutorMethodInterceptor

**AbstractJpaQuery.execute 执行具体的查询**

**语法解析在Part中**



