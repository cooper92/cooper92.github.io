---
layout: post
title: MyBatis-Plus教程
date: 2021-01-25
Author: 浪翻云
categories: 
tags: [MyBatis, Idea]
comments: true
--- 

# 概念
## 简介
官网：https://mybatis.plus/ 或 https://mp.baomidou.com/
码云地址：https://gitee.com/organizations/baomidou
MyBatis-Plus（简称 MP）是一个 MyBatis 的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生。

## 特性
*   无侵入：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑
*   损耗小：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作
*   强大的 CRUD 操作：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
*   支持 Lambda 形式调用：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错 
*   支持主键自动生成：支持多达 4 种主键策略(内含分布式唯一 ID 生成器 - Sequence)，可自由配置，完美解决主键问题
*   支持 ActiveRecord 模式：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强 大的 CRUD 操作
*   支持自定义全局通用操作：支持全局通用方法注入( Write once, use anywhere ) 
*   内置代码生成器：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用
*   内置分⻚插件：基于 MyBatis 物理分⻚，开发者无需关心具体操作，配置好插件之后，写分⻚等 同于普通 List 查询
*   分⻚插件支持多种数据库：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、 Postgre、SQLServer 等多种数据库
*   内置性能分析插件：可输出 Sql 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢 查询
*   内置全局拦截插件：提供全表 delete 、update 操作智能分析阻断，也可自定义拦截规则，预防 误操作

## 架构


# 快速入⻔
## 安装Spring Boot
    <dependency>
      <groupId>com.baomidou</groupId> 
      <artifactId>mybatis-plus-boot-starter</artifactId> 
      <version>3.4.0</version>
    </dependency>
## Spring MVC
    <dependency>
      <groupId>com.baomidou</groupId> 
      <artifactId>mybatis-plus</artifactId> 
      <version>3.4.0</version>
    </dependency>
    
对于Mybatis整合MP有常常有三种用法，分别是Mybatis+MP、Spring+Mybatis+MP、Spring Boot+Mybatis+MP。

## 创建数据库和表
test库
        -- 创建测试表
        DROP TABLE IF EXISTS user; 
        CREATE TABLE user
        (
            id BIGINT(20) NOT NULL COMMENT '主键ID',
            name VARCHAR(30) NULL DEFAULT NULL COMMENT '姓名', 
          age INT(11) NULL DEFAULT NULL COMMENT '年龄', 
          email VARCHAR(50) NULL DEFAULT NULL COMMENT '邮箱', 
          PRIMARY KEY (id)
        );
        -- 插入测试数据
        INSERT INTO user (id, name, age, email) VALUES 
        (1, 'Jone', 18, 'test1@baomidou.com'),
        (2, 'Jack', 20, 'test2@baomidou.com'),
        (3, 'Tom', 28, 'test3@baomidou.com'),
        (4, 'Sandy', 21, 'test4@baomidou.com'),
        (5, 'Billie', 24, 'test5@baomidou.com');
    
## 创建工程
导入依赖，并格式化
    <dependencies>
        <!-- mybatis-plus插件依赖 -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus</artifactId>
            <version>3.4.0</version>
        </dependency>
        <!--Mysql-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>
        <!--连接池-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.0.11</version>
        </dependency>
        <!--简化bean代码的工具包-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.4</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <dependency>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-api</artifactId>
                <version>1.7.5</version>
            </dependency>
            <dependency>
                <groupId>ch.qos.logback</groupId>
                <artifactId>logback-core</artifactId>
                <version>1.0.11</version>
            </dependency>
            <dependency>
                <groupId>ch.qos.logback</groupId>
                <artifactId>logback-classic</artifactId>
                <version>1.0.11</version>
            </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
    
## 用法1：Mybatis + MP
### 创建子Module 
banma-mybatis-plus-simple
### Mybatis实现查询User

**第一步，编写mybatis-config.xml文件:**

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
            PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
        <properties resource="jdbc.properties"></properties>
        <!--environments: 运行环境-->
        <environments default="development">
            <environment id="development">
                <!--当前的事务事务管理器是JDBC-->
                <transactionManager type="JDBC"></transactionManager>
                <!--数据源信息 POOLED:使用mybatis的连接池-->
                <dataSource type="POOLED">
                    <property name="driver" value="${jdbc.driver}"/>
                    <property name="url" value="${jdbc.url}"/>
                    <property name="username" value="${jdbc.username}"/>
                    <property name="password" value="${jdbc.password}"/>
                </dataSource>
            </environment>
        </environments>
        <!--引入映射配置文件-->
        <mappers>
            <mapper resource="mapper/UserMapper.xml"></mapper>
        </mappers>
    </configuration>
    
jdbc.properties：

    jdbc.driver=com.mysql.jdbc.Driver
    jdbc.url=jdbc:mysql:///test
    jdbc.username=root
    jdbc.password=admin
    
**第二步，编写User实体对象:(这里使用lombok进行了进化bean操作)**

    @Data // getter setter toString
    @NoArgsConstructor // 生成无参构造
    @AllArgsConstructor // 生成全参构造
    public class User {
        private Long id;
        private String name;
        private Integer age;
        private String email;
    }
    
**第三步，编写UserMapper接口:**

    public interface UserMapper {
        /**
         * 查询所有用户
         */
        public List<User> findAll();
    }
    
**第四步，编写UserMapper.xml文件:**

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
            PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.ebanma.cloud.mapper.UserMapper">
        <!--查询所有用户信息-->
        <select id="findAll" resultType="com.ebanma.cloud.entity.User">
            select * from user
        </select>
    </mapper>
    
**第五步，编写TestMybatis测试用例:**

    public class MPTest {
        @Test
        public void mybatisTest() throws IOException {
            InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
            SqlSession sqlSession = sqlSessionFactory.openSession();
            UserMapper mapper = sqlSession.getMapper(UserMapper.class);
            List<User> all = mapper.findAll();
            for (User user : all) {
                System.out.println(user);
            }
        }
    }
    
测试结果:

    User(id=1, name=Jone, age=18, email=test1@baomidou.com)
    User(id=2, name=Jack, age=20, email=test2@baomidou.com)
    User(id=3, name=Tom, age=28, email=test3@baomidou.com)
    User(id=4, name=Sandy, age=21, email=test4@baomidou.com)
    User(id=5, name=Billie, age=24, email=test5@baomidou.com)
    
### IDEA 小技巧
• 格式化快捷键：Ctrl+Alt+L
• 删除行快捷键：Ctrl+X
• .var 开启用推断类型替换var
• Declare final勾选且去不掉
• new一个对象
• 按下ctrl + alt +v
• 按下 ait + f
• 回车
• list.for
• sout
• psvm

### Mybatis+MP实现查询User
**第一步，将UserMapper继承BaseMapper，将拥有了BaseMapper中的所有方法:**

    public interface UserMapper extends BaseMapper<User> {
        /**
         * 查询所有用户
         */
        public List<User> findAll();
    }
    
**第二步，使用MP中的MybatisSqlSessionFactoryBuilder进程构建:**

    @Test
    public void mybatisTest2() throws IOException {
        InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");

        //这里使用的是MP中的MybatisSqlSessionFactoryBuilder
        SqlSessionFactory sqlSessionFactory = new MybatisSqlSessionFactoryBuilder().build(resourceAsStream);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);

        // 可以调用BaseMapper中定义的方法
        List<User> all = mapper.selectList(null);
        for (User user : all) {
            System.out.println(user);
        }
    }
    
> 注：如果实体类名称和表名称不一致，可以在实体类上添加注解@TableName("指定数据库表名") 
简单说明:
• 由于使用了 MybatisSqlSessionFactoryBuilder进行了构建，继承的BaseMapper中的方法就载入到了SqlSession中，所以就可以直接使用相关的方法;
如图：

## 用法2：Spring + Mybatis + MP
引入了Spring框架，数据源、构建等工作就交给了Spring管理。
### 创建子Module
banma-mybatis-plus-spring
### 导入依赖

    <properties>
        <spring.version>5.1.6.RELEASE</spring.version>
    </properties>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>${spring.version}</version>
        </dependency>
    </dependencies>
    
### 实现查询User
**第一步，编写jdbc.properties**

    jdbc.driver=com.mysql.jdbc.Driver
    jdbc.url=jdbc:mysql:///test
    jdbc.username=root
    jdbc.password=admin
    
**第二步，编写applicationContext.xml**

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:context="http://www.springframework.org/schema/context" xsi:schemaLocation="
    http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
        <context:property-placeholder location="classpath:jdbc.properties"></context:property-placeholder>
        <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
            <property name="driverClassName" value="${jdbc.driver}" />
            <property name="url" value="${jdbc.url}" />
            <property name="username" value="${jdbc.username}" />
            <property name="password" value="${jdbc.password}" />
        </bean>

        <!-- 1. 将sqlSessionFactory对象的创建交给spring-->
        <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
            <property name="dataSource" ref="dataSource" />
        </bean>
        <!-- 2.mapper映射扫描-->
        <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
            <property name="basePackage" value="com.ebanma.cloud.mapper" />
        </bean>
    </beans>
    
**替换为MP提供的sqlSessionFactory：**

      <!--这里使用MP提供的sqlSessionFactory,完成spring与mp的整合-->
          <bean id="sqlSessionFactory" class="com.baomidou.mybatisplus.extension.spring.MybatisSqlSessionFactoryBean">
          <property name="dataSource" ref="dataSource" />
      </bean>
      
**第三步，编写User对象以及UserMapper接口:**
前一个工程代码直接copy过来
**第四步，编写测试用例:**

    @RunWith(SpringJUnit4ClassRunner.class) 
    @ContextConfiguration(locations = "classpath:applicationContext.xml")
    public class TestSpringMP {
        @Autowired
        private UserMapper userMapper;
        @Test
        public void test2() throws IOException {
            List<User> users = this.userMapper.selectList(null); 
            for (User user : users) {
                System.out.println(user); 
            }
        }
    }
    
## 用法3：SpringBoot + Mybatis + MP
使用SpringBoot将进一步的简化MP的整合，需要注意的是，由于使用SpringBoot需要继承parent，所以需要重新创建工程，并不是创建子Module。
### 创建工程
banma-mp-springboot
### 导入依赖

    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>
        <parent>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>2.4.2</version>
            <relativePath/> <!-- lookup parent from repository -->
        </parent>
        <groupId>com.ebanma.cloud</groupId>
        <artifactId>banma-mp-springboot</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <name>banma-mp-springboot</name>
        <description>Demo project for Spring Boot</description>
        <properties>
            <java.version>1.8</java.version>
        </properties>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter</artifactId>
                <exclusions>
                    <exclusion>
                        <groupId>org.springframework.boot</groupId>
                        <artifactId>spring-boot-starter-logging</artifactId>
                    </exclusion>
                </exclusions>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
                <scope>test</scope>
            </dependency>
            <!--简化代码的工具包-->
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <optional>true</optional>
            </dependency>
            <!--mybatis-plus的springboot支持-->
            <dependency>
                <groupId>com.baomidou</groupId>
                <artifactId>mybatis-plus-boot-starter</artifactId>
                <version>3.4.0</version>
            </dependency>
            <!--mysql驱动-->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>5.1.47</version>
            </dependency>
        </dependencies>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                    <configuration>
                        <excludes>
                            <exclude>
                                <groupId>org.projectlombok</groupId>
                                <artifactId>lombok</artifactId>
                            </exclude>
                        </excludes>
                    </configuration>
                </plugin>
            </plugins>
        </build>
    </project>
    
### 编写application.properties

      spring.datasource.driver-class-name=com.mysql.jdbc.Driver
      spring.datasource.url=jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&characterEncoding=utf8&autoReconnect=true&allowMultiQueries=true&useSSL=false
      spring.datasource.username=root
      spring.datasource.password=zl3557884
      
### 编写entity

    @Data // getter setter toString
    @NoArgsConstructor // 生成无参构造
    @AllArgsConstructor // 生成全参构造
    public class User {
        private Long id;
        private String name;
        private Integer age;
        private String email;
    }
    
### 编写mapper

    public interface UserMapper extends BaseMapper<User> {
    }
    
### 编写启动类

    @MapperScan("com.ebanma.cloud.mapper")
    @SpringBootApplication
    public class BanmaMpSpringbootApplication {
        public static void main(String[] args) {
            SpringApplication.run(BanmaMpSpringbootApplication.class, args);
        }
    }
    
### 编写测试用例

    @SpringBootTest
    class BanmaMpSpringbootApplicationTests {
        @Autowired
        private UserMapper userMapper;
        @Test
        void contextLoads() {
            List<User> users = userMapper.selectList(null);
            for (User user : users) {
                System.out.println(user);
            }
        }
    }
