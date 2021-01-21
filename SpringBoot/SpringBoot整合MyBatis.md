## 简单配置

- mybatis.mapper-locations=classpath\*:mapper/\*\*/\*.xml
- mybatis.type-aliases-package=类型别名的包名
- mybatis.type-handlers-package=TypeHandler扫描报名
- mybatis.configuration.map-underscore-to-camel-case=true，把下划线转化成驼峰规则

## Mapper的定义与扫描

- @MapperScan( 配置扫描位置 )
- @Mapper定义接口
- 映射的定义——XML与注解

## 映射注解

- @Select

- @Insert
  - @Options(useGeneratedKeys = true)，主键回填
- @Delete
- @Update

**下面三个返回的都是影响的条数**

- @Results

demo如下

`schema.sql`

```sql
create table t_coffee (
    id bigint not null auto_increment,
    name varchar(255),
    price bigint not null,
    create_time timestamp,
    update_time timestamp,
    primary key (id)
);
```

`application.properties`

```properties
mybatis.type-handlers-package=com.jooks.springbootstudy03mybatis.handler
mybatis.configuration.map-underscore-to-camel-case=true
```

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class Coffee {
    private Long id;
    private String name;
    private Money price;
    private Date createTime;
    private Date updateTime;
}
```

```java
@Mapper
public interface CoffeeMapper {

    @Insert("insert into t_coffee (name, price, create_time, update_time)" +
            "values (#{name}, #{price}, now(), now())")
    @Options(useGeneratedKeys = true, keyColumn = "id", keyProperty = "id")
    int save(Coffee coffee);

    @Select("select * from t_coffee where id = #{id}")
    @Results({
            @Result(id = true, column = "id", property = "id"),
            @Result(column = "create_time", property = "createTime"),
            // map-underscore-to-camel-case = true 可以实现一样的效果
            // @Result(column = "update_time", property = "updateTime"),
    })
    Coffee findById(@Param("id") Long id);
}
```

```java
public class MoneyTypeHandler extends BaseTypeHandler<Money> {
    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, Money parameter, JdbcType jdbcType) throws SQLException {
        ps.setLong(i, parameter.getAmountMinorLong());
    }

    @Override
    public Money getNullableResult(ResultSet rs, String columnName) throws SQLException {
        return parseMoney(rs.getLong(columnName));
    }

    @Override
    public Money getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        return parseMoney(rs.getLong(columnIndex));
    }

    @Override
    public Money getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
        return parseMoney(cs.getLong(columnIndex));
    }

    private Money parseMoney(Long value) {
        return Money.of(CurrencyUnit.of("CNY"), value / 100.0);
    }
}
```

```java
@SpringBootApplication
@MapperScan("com.jooks.springbootstudy03mybatis.mapper")
@Slf4j
public class SpringbootStudy03MybatisApplication implements ApplicationRunner {
   @Autowired
   private CoffeeMapper coffeeMapper;

   public static void main(String[] args) {
      SpringApplication.run(SpringbootStudy03MybatisApplication.class, args);
   }

   @Override
   public void run(ApplicationArguments args) throws Exception {
      Coffee c = Coffee.builder().name("espresso")
            .price(Money.of(CurrencyUnit.of("CNY"), 20.0)).build();
      int count = coffeeMapper.save(c);
      log.info("Save {} Coffee: {}", count, c);

      c = Coffee.builder().name("latte")
            .price(Money.of(CurrencyUnit.of("CNY"), 25.0)).build();
      count = coffeeMapper.save(c);
      log.info("Save {} Coffee: {}", count, c);

      c = coffeeMapper.findById(c.getId());
      log.info("Find Coffee: {}", c);
   }
}
```