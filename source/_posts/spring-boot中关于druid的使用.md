---
title: spring boot中关于druid的使用
date: 2017-07-24 10:33:40
tags: [spring boot,druid]
categories: [java,spring boot,数据库]
---

#  Druid是什么
Druid是Java语言中最好的数据库连接池。Druid能够提供强大的监控和扩展功能。其中druid的github的仓库地址有很多关于druid集成的描述，如果需要详细的源码和介绍请前往**https://github.com/alibaba/druid**,本文主要详细介绍怎么将druid集成到spring boot的框架中，让其利用spring boot框架的优越性，更好地进行开发

---

# spring boot与druid的集成

##1.在 Spring Boot 项目中加入`druid-spring-boot-starter`依赖
```Maven
<dependency>
   <groupId>com.alibaba</groupId>
   <artifactId>druid-spring-boot-starter</artifactId>
   <version>1.1.2</version>
</dependency>
```
* 只要引入以上依赖包，便可以引入druid数据连接池，并且可以按照默认的配置自动装配，具体自动配置的类是`com.alibaba.druid.spring.boot.autoconfigure.DruidDataSourceAutoConfigure.java`，当检测到上下文不存在DataSource的实现类的时候，就会默认创建一个DruidDataSource，具体的实现代码如下：
```
    @Bean
    @ConditionalOnMissingBean
    public DataSource dataSource(DruidDataSourceProperties properties, Environment env) {
        DruidDataSource dataSource = DruidDataSourceBuilder.create().build(properties);
        if(dataSource.getUsername() == null) {
            dataSource.setUsername(env.getProperty("spring.datasource.username"));
        }

        if(dataSource.getPassword() == null) {
            dataSource.setPassword(env.getProperty("spring.datasource.password"));
        }

        if(dataSource.getUrl() == null) {
            dataSource.setUrl(env.getProperty("spring.datasource.url"));
        }

        if(dataSource.getDriverClassName() == null) {
            dataSource.setDriverClassName(env.getProperty("spring.datasource.driver-class-name"));
        }

        if(!"false".equals(env.getProperty("spring.datasource.druid.stat-view-servlet.enabled"))) {
            try {
                dataSource.setFilters("stat");
            } catch (SQLException var5) {
                var5.printStackTrace();
            }
        }
        return dataSource;
    }
```
---
##2.druid配置属性
Druid Spring Boot Starter 配置属性的名称完全遵照 Druid，你可以通过 Spring Boot 配置文件来配置Druid数据库连接池和监控，如果没有配置则使用默认值。

* JDBC 配置，提供了两个配置方式
1、使用**spring.datasource**
```
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/test
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.username=root
spring.datasource.password=123456
```
2、使用**spring.datasource.druid**
```
spring.datasource.druid.url=jdbc:mysql://127.0.0.1:3306/test
spring.datasource.druid.username=root
spring.datasource.druid.password=123456
spring.datasource.druid.driver-class-name=com.mysql.jdbc.Driver
```
---
* 连接池配置,主要是根据实际需要来配置druid连接池的设置，如果没有配置，则使用默认的配置，默认配置请看`com.alibaba.druid.pool.DruidAbstractDataSource.java`，如果需要手动配置，参考如下：
```
spring.datasource.druid.initial-size=  //初始化时建立物理连接的个数。初始化发生在显示调用init方法，或者第一次getConnection时
spring.datasource.druid.max-active=  //最大连接池数量
spring.datasource.druid.min-idle=   //最小连接池数量
spring.datasource.druid.max-wait=  //获取连接时最大等待时间，单位毫秒。配置了maxWait之后，缺省启用公平锁，并发效率会有所下降，如果需要可以通过配置useUnfairLock属性为true使用非公平锁。
spring.datasource.druid.pool-prepared-statements= //是否缓存preparedStatement，也就是PSCache。PSCache对支持游标的数据库性能提升巨大，比如说oracle。在mysql下建议关闭。 缺省false
spring.datasource.druid.max-pool-prepared-statement-per-connection-size=   //要启用PSCache，必须配置大于0，当大于0时，poolPreparedStatements自动触发修改为true。在Druid中，不会存在Oracle下PSCache占用内存过多的问题，可以把这个数值配置大一些，比如说100
spring.datasource.druid.max-open-prepared-statements= #和上面的等价
spring.datasource.druid.validation-query= //用来检测连接是否有效的sql，要求是一个查询语句。如果validationQuery为null，testOnBorrow、testOnReturn、testWhileIdle都不会其作用。
spring.datasource.druid.validation-query-timeout=//
spring.datasource.druid.test-on-borrow=//申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。
spring.datasource.druid.test-on-return=//归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能
spring.datasource.druid.test-while-idle=//	建议配置为true，不影响性能，并且保证安全性。申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效。
spring.datasource.druid.time-between-eviction-runs-millis= //有两个含义： 
//1) Destroy线程会检测连接的间隔时间2) testWhileIdle的判断依据，详细看testWhileIdle属性的说明
spring.datasource.druid.min-evictable-idle-time-millis=
spring.datasource.druid.max-evictable-idle-time-millis=
spring.datasource.druid.filters= #配置多个英文逗号分隔  //属性类型是字符串，通过别名的方式配置扩展插件，常用的插件有：//监控统计用的filter:stat日志用的filter:log4j防御sql注入的filter:wall
```
---

*  监控配置，druid提供了可视化的界面，用来监视数据库运行时的配置信息
```
WebStatFilter配置 WebStatFilter用于采集web-jdbc关联监控的数据。
spring.datasource.druid.web-stat-filter.enabled=true #是否启用StatFilter默认值true
spring.datasource.druid.web-stat-filter.url-pattern= /*
spring.datasource.druid.web-stat-filter.exclusions=*.js,*.gif,*.jpg,*.bmp,*.png,*.css,*.ico,/druid/*
spring.datasource.druid.web-stat-filter.session-stat-enable=
spring.datasource.druid.web-stat-filter.session-stat-max-count=
spring.datasource.druid.web-stat-filter.principal-session-name=
spring.datasource.druid.web-stat-filter.principal-cookie-name=
spring.datasource.druid.web-stat-filter.profile-enable=
```

```
# StatViewServlet配置，提供监控信息展示的html页面,提供监控信息的JSON API
spring.datasource.druid.stat-view-servlet.enabled=true #是否启用StatViewServlet默认值true
spring.datasource.druid.stat-view-servlet.url-pattern=/druid/*
spring.datasource.druid.stat-view-servlet.reset-enable=
spring.datasource.druid.stat-view-servlet.login-username=admin
spring.datasource.druid.stat-view-servlet.login-password=123456
spring.datasource.druid.stat-view-servlet.allow= //白名单
spring.datasource.druid.stat-view-servlet.deny=  //黑名单
```
##3.引入jpa依赖，用于操作数据库

* 当没有引入jpa或者mybaties等相关操作数据库的组件的时候，访问druid统计界面的时候，会没有任何的数据，只有当引入相关操作数据库的组件的时候，才会有相应的信息，其中引入jpa的配置如下
```
	<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
	</dependency>
```
* 当引入之后，启动应用，通过地址**http://localhost:8082/druid/index.html**访问，界面效果如下：

![druid访问界面效果](http://fantesycy-oss.oss-cn-shenzhen.aliyuncs.com/1.jpg)







