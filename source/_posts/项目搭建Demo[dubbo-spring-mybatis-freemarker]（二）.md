---
title: 项目搭建Demo[dubbo-spring-mybatis-freemarker]（二）
tags: [项目实践]
toc: true
---
# mybatis
## 新增mybatis依赖
数据库采用的是mysql
```
            <mybatis_version>3.2.1</mybatis_version>
            <mybatis_spring_version>1.2.0</mybatis_spring_version>
            <mysql-connector_version>5.1.39</mysql-connector_version>
            <c3p0_version>0.9.1.2</c3p0_version>
            <!-- MyBatis相关jar包 -->
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis</artifactId>
                <version>${mybatis_version}</version>
            </dependency>
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis-spring</artifactId>
                <version>${mybatis_spring_version}</version>
            </dependency>

            <!-- 数据库相关依赖 -->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>${mysql-connector_version}</version>
            </dependency>

            <!-- C3P0连接池 -->
            <dependency>
                <groupId>c3p0</groupId>
                <artifactId>c3p0</artifactId>
                <version>${c3p0_version}</version>
            </dependency>
```

## 配置spring的mybatis配置
### 配置数据源连接池
数据库配置文件
```
## 数据库配置文件
jdbc.user=root
jdbc.password=XXX
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/ELE_SHOP?useUnicode=true&characterEncoding=utf-8&useSSL=false
jdbc.minPoolSize=5
jdbc.maxPoolSize=10
jdbc.initialPoolSize=5
```
数据源配置   
```
    <!-- 导入属性配置文件 -->
    <context:property-placeholder location="classpath:database.properties" />

    <!-- 注解扫描 -->
    <!--<context:component-scan base-package="com.leegebe.userservice.service" />-->

    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource"
          destroy-method="close">
        <property name="driverClass" value="${jdbc.driverClassName}" />
        <property name="jdbcUrl" value="${jdbc.url}" />
        <property name="user" value="${jdbc.user}" />
        <property name="password" value="${jdbc.password}" />
        <!-- 保持最小连接数 -->
        <property name="minPoolSize" value="${jdbc.minPoolSize}" />
        <!-- 保持最大连接数 -->
        <property name="maxPoolSize" value="${jdbc.maxPoolSize}" />
        <!-- 初始化连接数 -->
        <property name="initialPoolSize" value="${jdbc.initialPoolSize}" />
        <!-- 连接的最大空闲时间，如果超过这个时间，某个数据库连接还没有被使用，则会断开掉这个连接,单位 s -->
        <property name="maxIdleTime" value="60" />
        <!-- #连接池在无空闲连接可用时一次性创建的新数据库连接数 -->
        <property name="acquireIncrement" value="5" />
        <!-- 连接池为数据源缓存的PreparedStatement的总数。由于PreparedStatement属于单个Connection,
        所以这个数量应该根据应用中平均连接数乘以每个连接的平均PreparedStatement来计算。
        同时maxStatementsPerConnection的配置无效。 -->
        <property name="maxStatements" value="0" />
        <!-- 用来配置测试空闲连接的间隔时间。测试方式还是上面的两种之一，可以用来解决MySQL8小时断开连接的问题。
            因为它保证连接池会每隔一定时间对空闲连接进行一次测试，
            从而保证有效的空闲连接能每隔一定时间访问一次数据库，
            将于MySQL8小时无会话的状态打破。 -->
        <property name="idleConnectionTestPeriod" value="60" />
        <!-- 连接池在获得新连接失败时重试的次数，如果小于等于0则无限重试直至连接获得成功。 -->
        <property name="acquireRetryAttempts" value="2" />
        <!--  如果为true，则当连接获取失败时自动关闭数据源，除非重新启动应用程序。所以一般不用。default : false（不建议使用） -->
        <property name="breakAfterAcquireFailure" value="false" />
        <!--  连接池在获得新连接时的间隔时间。单位ms-->
        <property name="acquireRetryDelay" value="1000"/>
        <property name="testConnectionOnCheckout" value="false" />
    </bean>
```
### 配置mybatis-config.xml
这里可以进行`typeAliases`和`mapper`等配置。
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--<typeAliases>-->
        <!--<typeAlias alias="StudentEntity" type="com.manager.data.model.StudentEntity"/>-->
    <!--</typeAliases>-->
    <!--<mappers>-->
        <!--<mapper resource="mapper/StudentMapper.xml" />-->
    <!--</mappers>-->
</configuration>
```

### 配置sqlSessionFactoryBean
mybatis的所有操作都是依赖SqlSession，这个工厂对象进行管理和创建。
```
    <!-- sqlSessionFactory -->
    <bean id="sqlSessionFactory" name="sqlSessionFactoryBean" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="configLocation" value="classpath:META-INF/spring/mybatis-config.xml" />
        <property name="dataSource" ref="dataSource" />
        <property name="mapperLocations">
            <list>
                <value>classpath*:META-INF/mapper/**/*Mapper.xml</value>
            </list>
        </property>
    </bean>
```

### 配置扫描mapper
```
    <!-- Mapper扫描配置 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.leegebe.userservice.dao" />
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactoryBean"></property>
    </bean>
```

## 配置事务
mybatis和spring集成后，mybatis自身的事务会被spring覆盖。
```
 <!-- 事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
    </bean>

    <!-- Transaction Configuration -->
    <bean id="matchAllTxInterceptor" class="org.springframework.transaction.interceptor.TransactionInterceptor">
        <property name="transactionManager">
            <ref bean="transactionManager" />
        </property>
        <property name="transactionAttributes">
            <props>
                <prop key="newT_*">PROPAGATION_REQUIRES_NEW</prop>
                <prop key="find*">PROPAGATION_REQUIRED,readOnly</prop>
                <prop key="get*">PROPAGATION_REQUIRED,readOnly</prop>
                <prop key="search*">PROPAGATION_REQUIRED,readOnly</prop>
                <prop key="query*">PROPAGATION_REQUIRED</prop>
                <prop key="*">PROPAGATION_REQUIRED</prop>
            </props>
        </property>
    </bean>

    <bean id="autoProxyCreator" class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator">
        <property name="interceptorNames">
            <list>
                <idref bean="matchAllTxInterceptor" />
            </list>
        </property>
        <property name="beanNames">
            <list>
                <value>*Api</value>
            </list>
        </property>
    </bean>
```

## 建立demo
### 建立Model对象User
```
public class User implements Serializable {

    private static final long serialVersionUID = -7396402169234330389L;

    private Long id;

    private String userId;

    private String userName;

    private String loginName;

    private String password;

    private Long createTime;

    private Long updateTime;

    private Integer isValidated;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getUserId() {
        return userId;
    }

    public void setUserId(String userId) {
        this.userId = userId;
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public String getLoginName() {
        return loginName;
    }

    public void setLoginName(String loginName) {
        this.loginName = loginName;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public Long getCreateTime() {
        return createTime;
    }

    public void setCreateTime(Long createTime) {
        this.createTime = createTime;
    }

    public Long getUpdateTime() {
        return updateTime;
    }

    public void setUpdateTime(Long updateTime) {
        this.updateTime = updateTime;
    }

    public Integer getIsValidated() {
        return isValidated;
    }

    public void setIsValidated(Integer isValidated) {
        this.isValidated = isValidated;
    }
}
```

### 建立mapper对象UserMapper
```
package com.leegebe.dubbo.mapper;

import com.leegebe.dubbo.model.User;

/**
 *
 */
public interface UserMapper {

    User getUserById(int id);

}
```
### 建立mapper文件
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.leegebe.dubbo.mapper.UserMapper">

    <resultMap id="userResultMap" type="com.leegebe.dubbo.model.User">
        <id column="ID" jdbcType="BIGINT" property="id" />
        <result column="USER_ID" jdbcType="VARCHAR" property="userId" />
        <result column="USER_NAME" jdbcType="VARCHAR" property="userName" />
        <result column="LOGIN_NAME" jdbcType="VARCHAR" property="loginName" />
        <result column="PASSWORD" jdbcType="VARCHAR" property="password" />
        <result column="CREATE_TIME" property="createTime" jdbcType="BIGINT" />
        <result column="UPDATE_TIME" property="updateTime" jdbcType="BIGINT" />
        <result column="IS_VALIDATE" property="isValidated" jdbcType="INTEGER" />
    </resultMap>

    <sql id="tableName">
        ele_shop_user
    </sql>

    <select id="getUserById" resultMap="userResultMap">
        SELECT * FROM <include refid="tableName"/> WHERE ID = #{id}
    </select>

</mapper>
```

### 建立serviceApi和impl
```
package com.leegebe.dubbo.api;

import com.leegebe.dubbo.model.User;

/**
 * 查询用户API
 */
public interface UserApi {

    User getUserById(int id);

}
```

```
package com.leegebe.dubbo.api.impl;

import com.leegebe.dubbo.api.UserApi;
import com.leegebe.dubbo.mapper.UserMapper;
import com.leegebe.dubbo.model.User;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;

/**
 * User接口实现类
 */
@Service("userApi")
public class UserApiImpl implements UserApi {

    @Resource
    UserMapper userMapper;

    public User getUserById(int id) {
        return userMapper.getUserById(id);
    }
}

```
### 建立测试类
```
package com.leegebe.dubbo.api.impl;

import com.leegebe.dubbo.api.UserApi;
import com.leegebe.dubbo.model.User;
import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import javax.annotation.Resource;

import static org.junit.Assert.*;

/**
 *
 */
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations={"classpath*:META-INF/spring/*.xml"})
public class UserApiImplTest {

    @Autowired
    UserApi userApi;

    @Test
    public void getUserById() throws Exception {
        int userId = 1;
        User user = userApi.getUserById(userId);
        Assert.assertNotNull(user);
        System.out.println(user.getUserName());
    }

}
```

### 前后端连调测试
```
 <dubbo:service interface="com.leegebe.dubbo.api.UserApi" ref="userApi" version="${dubbo.service.version}"/>
```

```
    <!-- 生成远程服务代理，可以和本地bean一样使用 -->
    <dubbo:reference id="userApi" interface="com.leegebe.dubbo.api.UserApi" version="${dubbo.service.version}"/>
```
版本号在配置文件中定义并在spring中引入即可。

# 总结
搭建过程中，遇到一些问题，总结如下。
## IDEA新建子项目问题
`pom.xml already exists in vfs`，原因是在建立是Content root和Module file location的配置不正确，选择了父项目的目录，当前父项目中已经有了pom文件，所以会提示错误。

## tomcat部署时ClassNotFound
是因为在部署的时候没有部署maven依赖的包，Project Structure->Artifacts->Output Layout，将Available Elements下的maven依赖，以及dubbo-service-api依赖添加到部署目录的lib包下。

## java.lang.IllegalStateException: Could not load TestContextBootstrapper [null]
原因是spring-test依赖的版本和spring的版本不一致导致

## 配置文件导入问题
```
    <context:property-placeholder location="classpath:database.properties,classpath:conf.properties" />
```
如果在俩个文件中分别引入配置文件导入，会导致一个失效。