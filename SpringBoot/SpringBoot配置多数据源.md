#### 方法一：

配置@Primary类型的Bean

#### 方法二：

排除(exclude)Spring Boot的自动配置：

- DataSourceAutoConfiguration
- DataSourceTransactionManagetAutoConfiguration
- JdbcTemplateAutoConfiguration

配置文件：

```properties
management.endpoints.web.exposure.include=*
spring.output.ansi.enabled=ALWAYS

foo.datasource.url=jdbc:h2:men:foo
foo.datasource.username=sa
foo.datasource.password=

bar.datasource.url=jdbc:h2:men:bar
bar.datasource.username=sa
bar.datasource.password=
```

java程序：

```java
@SpringBootApplication(exclude = {
		DataSourceAutoConfiguration.class,
		DataSourceTransactionManagerAutoConfiguration.class,
		JdbcTemplateAutoConfiguration.class
})
@Slf4j
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}

	@Bean
	@ConfigurationProperties("foo.datasource")
	public DataSourceProperties fooDataSourceProperties() {
		return new DataSourceProperties();
	}

	@Bean
	public DataSource fooDataSource() {
		DataSourceProperties dataSourceProperties = fooDataSourceProperties();
		//log.info("foo datasource: {}", dataSourceProperties.getUrl());
		return dataSourceProperties.initializeDataSourceBuilder().build();
	}

	@Bean
	@Resource
	public DataSourceTransactionManager fooDataSourceTransactionManager(DataSource fooDataSource) {
		return new DataSourceTransactionManager(fooDataSource);
	}

	@Bean
	@ConfigurationProperties("bar.datasource")
	public DataSourceProperties barDataSourceProperties() {
		return new DataSourceProperties();
	}

	@Bean
	public DataSource barDataSource() {
		DataSourceProperties dataSourceProperties = barDataSourceProperties();
		//log.info("bar datasource: {}", dataSourceProperties.getUrl());
		return dataSourceProperties.initializeDataSourceBuilder().build();
	}

	@Bean
	@Resource
	public DataSourceTransactionManager barDataSourceTransactionManager(DataSource barDataSource) {
		return new DataSourceTransactionManager(barDataSource);
	}
}
```

