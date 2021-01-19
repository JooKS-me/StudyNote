spring-jdbc包中含四个部分：

- core，JdbcTemplate等核心接口和类
- datasource，数据源相关的辅助类
- object，讲基本的JDBC操作封装
- support，错误码等其他辅助工具

> Spring中，数据库操作对应的Bean注解应该为@Repository

#### 1. spring-jdbc单sql语句操作方法

首先，定义一个实体类Foo（这里使用了lombok）

```java
@Data
@Builder
public class Foo {
    private Long id;
    private String bar;
}
```

然后是Dao

```java
@Slf4j
@Repository
public class FooDao {
    @Autowired
    private JdbcTemplate jdbcTemplate;
    @Autowired
    private SimpleJdbcInsert simpleJdbcInsert;

    public void insertData() {
        // insert方法一
        Arrays.asList("b", "c").forEach(bar -> {
            jdbcTemplate.update("INSERT INTO FOO (BAR) VALUES (?)", bar);
        });
		
        // insert方法二，利用了SimpleJdbcInsert类，Application.java中将SimpleJdbcInsert与表Foo和主键ID进行了绑定，代码在下面给出
        HashMap<String, String> row = new HashMap<>();
        row.put("BAR", "d");
        Number id = simpleJdbcInsert.executeAndReturnKey(row); //拿到返回的主键
        log.info("ID of d: {}", id.longValue());
    }

    public void listData() {
        // query for count
        log.info("Count: {}",
                jdbcTemplate.queryForObject("SELECT COUNT(*) FROM FOO", Long.class));

        // query到一群数据(每项只有一个)，存放在list中
        List<String> list = jdbcTemplate.queryForList("SELECT BAR FROM FOO", String.class);
        list.forEach(s -> log.info("Bar: {}", s)); //遍历输出list

        // query到一群数据(每项有多个)，将每项存成一个Foo对象，最终汇总到一个list中
        List<Foo> fooList = jdbcTemplate.query("SELECT * FROM FOO", new RowMapper<Foo>() {
            @Override
            public Foo mapRow(ResultSet rs, int rowNum) throws SQLException {
                return Foo.builder()
                        .id(rs.getLong(1))
                        .bar(rs.getString(2))
                        .build(); //将查询到的一个结果存放在一个Foo对象中
            }
        });
        fooList.forEach(f -> log.info("Foo: {}", f)); //遍历输出list
    }
}
```

```java
    @Bean
    @Autowired
    public SimpleJdbcInsert simpleJdbcInsert(JdbcTemplate jdbcTemplate) {
        return new SimpleJdbcInsert(jdbcTemplate)
                .withTableName("FOO").usingGeneratedKeyColumns("ID");
    }
```

#### 2. 执行sql批量操作

```java
@Repository
public class BatchFooDao {
    @Autowired
    private JdbcTemplate jdbcTemplate;
    @Autowired
    private NamedParameterJdbcTemplate namedParameterJdbcTemplate;

    public void batchInsert() {
        // 方法一：使用JdbcTemplate类
        jdbcTemplate.batchUpdate("INSERT INTO FOO (BAR) VALUES (?)",
                new BatchPreparedStatementSetter() {
                    @Override
                    public void setValues(PreparedStatement ps, int i) throws SQLException {
                        ps.setString(1, "b-" + i); //最终会插入b-1, b-2两条数据
                    }

                    // 返回批量sql操作次数
                    @Override
                    public int getBatchSize() {
                        return 2;
                    }
                });

        // 方法二：使用NamedParameterJdbcTemplate类
        List<Foo> list = new ArrayList<>();
        list.add(Foo.builder().id(100L).bar("b-100").build());
        list.add(Foo.builder().id(101L).bar("b-101").build());
        namedParameterJdbcTemplate
                .batchUpdate("INSERT INTO FOO (ID, BAR) VALUES (:id, :bar)",
                        SqlParameterSourceUtils.createBatch(list));
    }
}
```

