## 法一

首先，写好entity类、dao接口、dao接口的xml配置。

（略）

然后，配置spring-dao.xml，写好DataSource、SqlSessionFactory、SqlSessionTemplate(SqlSession)的bean

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/companydb?useSSL=false"/>
        <property name="username" value="root"/>
        <property name="password" value="123456"/>
    </bean>
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" value="mybatis-config.xml"/>
        <property name="mapperLocations" value="com/jooks/dao/*.xml"/>
    </bean>
    <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg index="0" ref="sqlSessionFactory"/>
    </bean>
</beans>
```

然后，编写DaoImpl类，并通过applicationContext.xml配置将上面得到sqlSession注入DaoImpl

```java
public class UserDaoImpl implements UserDao {
    private SqlSessionTemplate sqlSession;

    public void setSqlSession(SqlSessionTemplate sqlSession) {
        this.sqlSession = sqlSession;
    }

    public List<User> queryAll() {
        UserDao userDao = sqlSession.getMapper(UserDao.class);
        return userDao.queryAll();
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <import resource="spring-dao.xml"/>

    <bean id="userDao" class="com.jooks.dao.UserDaoImpl">
        <property name="sqlSession" ref="sqlSession"/>
    </bean>
</beans>
```

最后，使用DaoImpl即可。

## 法二：继承SqlSessionDaoSupport

首先，写好entity类、dao接口、dao接口的xml配置。

（略）

然后，配置spring-dao.xml，写好DataSource、SqlSessionFactory的bean。

然后，编写好DaoImpl（需要继承SqlSessionDaoSupport，然后通过getSqlSession方式获取sqlSession），并通过xml配置将sqlSessionFactoray注入DaoImpl

```java
public class UserDaoImpl2 extends SqlSessionDaoSupport implements UserDao {
    public List<User> queryAll() {
        return getSqlSession().getMapper(UserDao.class).queryAll();
    }
}
```

```xml
    <bean id="userDao2" class="com.jooks.dao.UserDaoImpl2">
        <property name="sqlSessionFactory" ref="sqlSessionFactory"/>
    </bean>
```

最后，使用DaoImpl即可。