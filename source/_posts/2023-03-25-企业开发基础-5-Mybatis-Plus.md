---
title: 企业开发基础-5-Mybatis-Plus
date: 2023-03-25 17:59:52
tags: 
  - MyBatis
categories: 
  - Technology
---

# 简介

> 概述

MyBatis-Plus 是一个 MyBatis 的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生。 

> 特性

* **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑 
* **损耗小**：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作 
* **强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求 
* **支持 Lambda 形式调用**：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错 
* **支持主键自动生成**：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题 
* **支持 ActiveRecord 模式**：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强 大的 CRUD 操作 
* **支持自定义全局通用操作**：支持全局通用方法注入（ Write once, use anywhere ） 
* **内置代码生成器**：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用 
* **内置分页插件**：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等 同于普通 List 查询 
* **分页插件支持多种数据库**：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、 Postgre、SQLServer 等多种数据库 
* **内置性能分析插件**：可输出 SQL 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出 慢查询 
* **内置全局拦截插件**：提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防 误操作  

> 框架结构

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/03-mybatis-plus-20230325-01.png)

# 快速入门

## 环境准备

| 环境         | 说明        |
| ------------ | ----------- |
| IDE          | IDEA 2018   |
| JDK          | Java 8      |
| 项目管理工具 | maven 3.6.2 |
| 数据库版本   | MySQL 5.6   |
| Spring Boot  | 2.6.3       |
| Mybatis plus | 3.5.1       |

## 数据库准备

```sql
CREATE DATABASE `mybatis_plus` /*!40100 DEFAULT CHARACTER SET utf8mb4 */;
use `mybatis_plus`;
CREATE TABLE `user` (
`id` bigint(20) NOT NULL COMMENT '主键ID',
`name` varchar(30) DEFAULT NULL COMMENT '姓名',
`age` int(11) DEFAULT NULL COMMENT '年龄',
`email` varchar(50) DEFAULT NULL COMMENT '邮箱',
PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

```sql
INSERT INTO user (id, name, age, email) VALUES
(1, 'Jone', 18, 'test1@baomidou.com'),
(2, 'Jack', 20, 'test2@baomidou.com'),
(3, 'Tom', 28, 'test3@baomidou.com'),
(4, 'Sandy', 21, 'test4@baomidou.com'),
(5, 'Billie', 24, 'test5@baomidou.com');
```

## 项目搭建

> **引入 SpringBootStarter 父工程**

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5+ 版本</version>
    <relativePath/>
</parent>
```

```xml
  <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.5.1</version>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.8</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.6</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>2.0.5</version>
        </dependency>
    </dependencies>
```

> 配置文件

```yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mybatis_plus?characterEncoding=utf8&useSSL=false
    username: XXX
    password: XXX
    driver-class-name: com.mysql.cj.jdbc.Driver
```

| driver-class-name                          | 说明                                        |
| ------------------------------------------ | ------------------------------------------- |
| spring boot **2.0**（内置jdbc5驱动）       | driver-class-name: com.mysql.jdbc.Driver    |
| spring boot **2.1及以上**（内置jdbc8驱动） | driver-class-name: com.mysql.cj.jdbc.Driver |

| url              | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| MySQL**5.7版本** | jdbc:mysql://localhost:3306/mybatis_plus?characterEncoding=utf-8&useSSL=false |
| MySQL**8.0版本** | jdbc:mysql://localhost:3306/mybatis_plus? serverTimezone=GMT%2B8&characterEncoding=utf-8&useSSL=false |

在 Spring Boot 启动类中添加 `@MapperScan` 注解，扫描 Mapper 文件夹： 

```java
@SpringBootApplication
@MapperScan("com.cyan.mp.mapper")
public class MyBatisPlusMainApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyBatisPlusMainApplication.class, args);
    }
}
```

## 编写代码

> 实体类

```java
@Data
public class User {

    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```

> mapper

```java
public interface UserMapper extends BaseMapper<User> {
}
```

## 开始使用

> 添加日志

```yml
# 配置MyBatis日志
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

> 添加测试类

```java
@SpringBootTest
public class UserMapperTest {

    @Resource
    private UserMapper userMapper;

    @Test
    public void testSelect() {
        List<User> userList = userMapper.selectList(null);

        userList.forEach(System.out::println);

    }
}
```

# CRUD

## BaseMapper

MyBatis-Plus中的基本CRUD在内置的BaseMapper中都已得到了实现，我们可以直接使用，接口如下：

```java
public interface BaseMapper<T> extends Mapper<T> {
```

### insert

| 方法                  | 参数     | 说明                     |
| --------------------- | -------- | ------------------------ |
| `int insert(T entity);` | 实体对象 | 根据实体对象插入一条记录 |

```java
/**
* 插入一条记录
* @param entity 实体对象
*/
int insert(T entity);
```

### delete

| 方法                                                         | 参数                                                         | 说明                          |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ----------------------------- |
| `int deleteById(Serializable id);`                             | 主键ID                                                       | 根据 ID 删除                  |
| `int deleteById(T entity);`                                    | 实体对象                                                     | 根据实体(ID)删除              |
| `int deleteByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);` | 表字段 map 对象                                              | 根据 columnMap 条件，删除记录 |
| `int delete(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);` | 实体对象封装操作类（可以为 null,里面的 entity 用于生成 where | 根据 entity 条件，删除记录    |
| `int deleteBatchIds(@Param(Constants.COLLECTION) Collection<? extends<br/>Serializable> idList);` | 主键ID列表(不能为 null 以及 empty)                           | 删除（根据ID 批量删除）       |



```java
/**
* 根据 ID 删除
* @param id 主键ID
*/
int deleteById(Serializable id);
```

```java
/**
* 根据实体(ID)删除
* @param entity 实体对象
* @since 3.4.4
*/
int deleteById(T entity);
```

```java
/**
* 根据 columnMap 条件，删除记录
* @param columnMap 表字段 map 对象
*/
int deleteByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);
```

```java
/**
* 根据 entity 条件，删除记录
* @param queryWrapper 实体对象封装操作类（可以为 null,里面的 entity 用于生成 where
语句）
*/
int delete(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
```

```java
/**
* 删除（根据ID 批量删除）
* @param idList 主键ID列表(不能为 null 以及 empty)
*/
int deleteBatchIds(@Param(Constants.COLLECTION) Collection<? extends
Serializable> idList);
```

### update

| 方法                                                         | 参数                                                         | 说明                            |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------- |
| `int updateById(@Param(Constants.ENTITY) T entity);`           | 实体对象                                                     | 根据 ID 修改                    |
| `int update(@Param(Constants.ENTITY) T entity, @Param(Constants.WRAPPER)Wrapper<T> updateWrapper);` | entity 实体对象 (set 条件值,可以为 null)；updateWrapper 实体对象封装操作类（可以为 null,里面的 entity 用于生成 | 根据 whereEntity 条件，更新记录 |

```java
/**
* 根据 ID 修改
* @param entity 实体对象
*/
int updateById(@Param(Constants.ENTITY) T entity);
```

```java
/**
* 根据 whereEntity 条件，更新记录
* @param entity 实体对象 (set 条件值,可以为 null)
* @param updateWrapper 实体对象封装操作类（可以为 null,里面的 entity 用于生成
where 语句）
*/
int update(@Param(Constants.ENTITY) T entity, @Param(Constants.WRAPPER)
Wrapper<T> updateWrapper);
```

```java
/**
* 根据 ID 查询
* @param id 主键ID
*/
T selectById(Serializable id);
```

### select

| 方法                                                         | 参数                                                         | 说明                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------------------- |
| `List<T> selectBatchIds(@Param(Constants.COLLECTION) Collection<? extendsSerializable> idList)` | 主键ID列表(不能为 null 以及 empty)                           | 查询（根据ID 批量查询）                  |
| `List<T> selectByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);` | 表字段 map 对象                                              | 查询（根据 columnMap 条件）              |
| `T selectOne(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper)` | 实体对象封装操作类（可以为 null）                            | 根据 entity 条件，查询一条记录           |
| `boolean exists(Wrapper<T> queryWrapper)`                    | 实体对象封装操作类                                           | 根据 Wrapper 条件，判断是否存在记录      |
| `Long selectCount(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);` | 实体对象封装操作类（可以为 null）                            | 根据 Wrapper 条件，查询总记录数          |
| `List<T> selectList(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);` | 实体对象封装操作类（可以为 null                              | 根据 entity 条件，查询全部记录           |
| `List<Map<String, Object>> selectMaps(@Param(Constants.WRAPPER) Wrapper<T>queryWrapper);` | 实体对象封装操作类（可以为 null                              | 根据 Wrapper 条件，查询全部记录          |
| `List<Object> selectObjs(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);` | 实体对象封装操作类（可以为 null                              | 根据 Wrapper 条件，查询全部记录          |
| `<P extends IPage<T>> P selectPage(P page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);` | page 分页查询条件（可以为 RowBounds.DEFAULT）;queryWrapper 实体对象封装操作类（可以为 null） | 根据 entity 条件，查询全部记录（并翻页） |
| `<P extends IPage<Map<String, Object>>> P selectMapsPage(P page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);` | page 分页查询条件;queryWrapper 实体对象封装操作类（可以为 null | 根据 Wrapper 条件，查询全部记录（并翻页  |

```java
/**
* 查询（根据ID 批量查询）
* @param idList 主键ID列表(不能为 null 以及 empty)
*/
List<T> selectBatchIds(@Param(Constants.COLLECTION) Collection<? extends
Serializable> idList);
```

```java
/**
* 查询（根据 columnMap 条件）
* @param columnMap 表字段 map 对象
*/
List<T> selectByMap(@Param(Constants.COLUMN_MAP) Map<String, Object>
columnMap);
```

```java
/**
* 根据 entity 条件，查询一条记录
* <p>查询一条记录，例如 qw.last("limit 1") 限制取一条记录, 注意：多条数据会报异常
</p>
* @param queryWrapper 实体对象封装操作类（可以为 null）
*/
default T selectOne(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper) {
List<T> ts = this.selectList(queryWrapper);
if (CollectionUtils.isNotEmpty(ts)) {
if (ts.size() != 1) {
throw ExceptionUtils.mpe("One record is expected, but the query
result is multiple records");
}
return ts.get(0);
}
return null;
}
```

```java
/**
 * 根据 Wrapper 条件，判断是否存在记录
 *
 * @param queryWrapper 实体对象封装操作类
 * @return
 */
default boolean exists(Wrapper<T> queryWrapper) {
    Long count = this.selectCount(queryWrapper);
    return null != count && count > 0;
}
```

```java
/**
* 根据 Wrapper 条件，查询总记录数
* @param queryWrapper 实体对象封装操作类（可以为 null）
*/
Long selectCount(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
```

```java
/**
* 根据 entity 条件，查询全部记录
* @param queryWrapper 实体对象封装操作类（可以为 null）
*/
List<T> selectList(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
```

```java
/**
* 根据 Wrapper 条件，查询全部记录
* @param queryWrapper 实体对象封装操作类（可以为 null）
*/
List<Map<String, Object>> selectMaps(@Param(Constants.WRAPPER) Wrapper<T>
queryWrapper);
```

```java
/**
* 根据 Wrapper 条件，查询全部记录
* <p>注意： 只返回第一个字段的值</p>
* @param queryWrapper 实体对象封装操作类（可以为 null）
*/
List<Object> selectObjs(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
```

```java
/**
* 根据 entity 条件，查询全部记录（并翻页）
* @param page 分页查询条件（可以为 RowBounds.DEFAULT）
* @param queryWrapper 实体对象封装操作类（可以为 null）
*/
<P extends IPage<T>> P selectPage(P page, @Param(Constants.WRAPPER)
Wrapper<T> queryWrapper);
```

```java
/**
* 根据 Wrapper 条件，查询全部记录（并翻页）
* @param page 分页查询条件
* @param queryWrapper 实体对象封装操作类
*/
<P extends IPage<Map<String, Object>>> P selectMapsPage(P page,
@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
```

## 插入操作

```java
@Test
public void testInsert() {
    /**
     * INSERT INTO user ( id, name, age, email ) VALUES ( ?, ?, ?, ? )
     */
    User user = new User(null, "张三", 23, "zhangsan@cyan.com");

    int ans = userMapper.insert(user);

    System.out.println("an = " + ans);

    System.out.println("id = " + user.getId());

}
```

雪花算法生成 id

## 删除操作

### deleteById

```java
@Test
public void testDeleteById() {

    // DELETE FROM user WHERE id=?
    int ans = userMapper.deleteById(5);
    System.out.println("an = " + ans);
}
```

### deleteBatchIds

```java
@Test
public void testDeleteBatchIds(){

    // DELETE FROM user WHERE id IN ( ? , ? , ? )
    List<Long> idList = Arrays.asList(1L, 2L, 3L);
    int ans = userMapper.deleteBatchIds(idList);
    System.out.println("an = " + ans);
}
```

### deleteByMap

```java
@Test
public void testDeleteByMap(){

    // DELETE FROM user WHERE name = ? AND age = ?
    Map<String, Object> map = new HashMap<>();
    map.put("age", 23);
    map.put("name", "张三");

    int ans = userMapper.deleteByMap(map);
    System.out.println("an = " + ans);
}
```

## 修改操作

```java
@Test
public void testUpdateById(){
    // UPDATE user SET name=?, age=? WHERE id=?
    User user = new User(4L, "admin", 22, null);
    int ans  = userMapper.updateById(user);
    System.out.println("ans  = " + ans);
}
```

## 查询操作

### selectById

```java
@Test
public void testSelectById(){
    // SELECT id,name,age,email FROM user WHERE id=?
    User user = userMapper.selectById(4L);
    System.out.println(user);
}
```

### selectBatchIds

```java
@Test
public void testSelectBatchIds(){
    // SELECT id,name,age,email FROM user WHERE id IN ( ? , ? )
    List<Long> idList = Arrays.asList(4L, 5L);

    List<User> list = userMapper.selectBatchIds(idList);
    list.forEach(System.out::println);
}
```

### selectByMap

```java
@Test
public void testSelectByMap(){
    // SELECT id,name,age,email FROM user WHERE name = ? AND age = ?
    Map<String, Object> map = new HashMap<>();
    map.put("age", 22);
    map.put("name", "admin");

    List<User> list = userMapper.selectByMap(map);
    // list.forEach(System.out::println);
    list.forEach(user -> System.out.println());
}
```

### selectList

```java
@Test
public void testSelectList(){
    // SELECT id,name,age,email FROM user
    List<User> list = userMapper.selectList(null);
    list.forEach(System.out::println);
}
```

通过观察 BaseMapper 中的方法，大多方法中都有 **Wrapper类型的形参，此为条件构造器**，可针对于 SQL 语句设置不同的条件，若没有条件，则可以为该形参赋值 null，即查询（删除/修改）所有数据  

## 通用 Service

MyBatis-Plus 中有一个接口 IService 和其实现类 ServiceImpl，封装了常见的业务层逻辑

> 创建 Service 接口和实现类

```java
// UserService继承IService模板提供的基础功能
public interface UserService extends IService<User> {
}
```

```java
// ServiceImpl实现了IService，提供了IService中基础功能的实现
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
}
```

> 测试查询记录数

```java
@Test
public void testGetCount(){

    long count = userService.count();
    System.out.println("总记录数：" + count);
}
```

> 测试批量插入

```java
@Test
public void testSaveBatch(){
    // SQL长度有限制，海量数据插入单条SQL无法实行，
    // 因此MP将批量插入放在了通用Service中实现，而不是通用Mapper

    ArrayList<User> users = new ArrayList<>();

    for (int i = 0; i < 6; i++) {
        User user = new User();
        user.setName("cyanzzy" + i);
        user.setAge(20 + i);
        users.add(user);
    }
    // INSERT INTO t_user ( username, age ) VALUES ( ?, ? )
    userService.saveBatch(users);
}
```

MyBatis-Plus 在确定操作的表时，由 `BaseMapper` 的泛型决定，即 **实体类型决定**，且 **默认操作的表名和实体类型的类名一致**  

# MP常用注解

## @TableName 

> 若 **实体类类型的类名和要操作的表的表名不一致**，会出现什么问题 

**将表user更名为t_user**，测试查询功能程序抛出异常，`Table 'mybatis_plus.user' doesn't exist`，因为现在的表名为t_user，而**默认操作的表名和实体类型的类名一致**，即user表

> 解决方案1 **@TableName** 

在实体类类型上添加`@TableName("t_user")`，标识实体类对应的表，即可成功执行SQL语句 

```java
@TableName("t_user")
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```

> 解决方案2 **全局配置**

当遇到实体类对应的表带有固定前缀时，使用 MyBatis-Plus 提供的 **全局配置**，为实体类所对应的表名设置默认的前缀，那么就不需要在每个实体类上通过 @TableName 标识实体类对应的表  

```yml
mybatis-plus:
  configuration:
    # 配置MyBatis日志
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  global-config:
    db-config:
      # 配置MyBatis-Plus操作表的默认前缀
      table-prefix: t_
```

## @TableId

经过以上的测试，MyBatis-Plus 在实现 CRUD 时，会默认将 id 作为主键列，并在插入数据时，**默认基于雪花算法的策略生成 id** 

> 若 **实体类和表中表示主键的不是 id，而是其他字段**，例如uid，MPs会自动识别uid为主键列吗？ 

实体类中的属性 id 改为 uid，将表中的字段 id 也改为 uid，测试添加功能  程序抛出异常，`Field 'uid' doesn't have a default value`，说明 MyBatis-Plus 没有将 uid 作为主键赋值

> 解决方案

在实体类中 uid 属性上通过 @TableId **将其标识为主键**，即可成功执行 SQL 语句 

```java
public class User {
    @TableId
    private Long uid;
    private String name;
    private Integer age;
    private String email;
}
```

### value

若实体类中主键对应的属性为 **id**，而表中表示主键的字段为 **uid**，此时**若只在属性 id 上添加注解** @TableId，则抛出异常`Unknown column 'id' in 'field list'`，即 MyBatis-Plus **仍然会将id作为表的主键** 操作

而表中表示主键的是字段 uid 此时需要通过 `@TableId` 注解的value属性，**指定表中的主键字段**

```java
@TableId("uid")或 @TableId(value="uid") 
```

### type

>  type属性用来定义主键策略  

| 值                        | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| IdType.ASSIGN_ID（默认） | 基于**雪花算法**的策略生成数据id，与数据库id是否设置自增无关 |
| IdType.AUTO               | 使用数据库的**自增策略**，注意，该类型请**确保**数据库设置了id自增， 否则无效 |

```yml
mybatis-plus:
  configuration:
    global-config:
      db-config:
        # 配置MyBatis-Plus操作表的默认前缀
        table-prefix: t_
        # 配置MyBatis-Plus的主键策略
        id-type: auto
```

## @TableField 

经过以上的测试，我们可以发现，MyBatis-Plus 在执行 SQL 语句时，要保证实体类中的属性名和表中的字段名一致 

> **如果实体类中的属性名和字段名不一致的情况**，会出现什么问题呢？ 

> 解决方案1 **开启驼峰命名映射**

若 **实体类中的属性使用的是驼峰命名风格，而表中的字段使用的是下划线命名风格** 

例如实体类属性 userName，表中字段 user_name 此时 MyBatis-Plus会 **自动** 将下划线命名风格 **转化为驼峰命名风格** 相当于在 MyBatis 中配置 

> 解决方案2 **TableField**

若 **实体类中的属性和表中的字段不满足解决方案1情况** 

例如实体类属性 name，表中字段 username 此时需要在实体类属性上 **使用 @TableField("username") 设置属性所对应的字段名** 

```java
public class User {
    private Long id;
    @TableField("username")
    private String name;
    private Integer age;
    private String email;
}
```

## @TableLogic 

> 逻辑删除 

* 物理删除：**真实删除**，将对应数据从数据库中删除，之后查询不到此条被删除的数据 
* 逻辑删除：假删除，将对应数据中代表是否被删除字段的 **状态修改** 为“被删除状态”，之后在数据库中仍旧能看到此条数据记录 
* 使用场景：可以进行 **数据恢复** 

> 解决方案

1.数据库创建逻辑删除状态列，设置默认值 0

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/03-mybatis-plus-20230325-04.png)

2.实体类中添加逻辑删除属性

```java
public class User {

    private Long id;
    private String name;
    private Integer age;
    private String email;
    @TableLogic
    private Integer isDeleted;
}
```

3.测试删除功能，真正执行的是修改 

```sql
UPDATE t_user SET is_deleted=1 WHERE id=? AND is_deleted=0 
```

4.测试查询功能，被逻辑删除的数据默认不会被查询 

```sql
SELECT id,username AS name,age,email,is_deleted FROM t_user WHERE is_deleted=0 
```

# 雪花算法

## 背景

需要选择合适的方案去**应对**数据规模的**增长**，以**应对**逐渐增长的**访问压力和数据量**。

数据库的扩展方式主要包括：**业务分库、主从复制，数据库分表。**  

## 数据库分表

将不同业务数据分散存储到不同的数据库服务器，能够支撑百万甚至千万用户规模的业务，但如果业务继续发展，同一业务的单表数据也会达到单台数据库服务器的处理瓶颈。

 单表数据拆分有两种方式：**垂直分表和水平分表**。示意图如下：  

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/03-mybatis-plus-20230325-02.png)

### 垂直分表 

垂直分表适合将表中**某些不常用且占了大量空间的列**拆分出去。

例如，前面示意图中的 nickname 和 description 字段，假设我们是一个婚恋网站，用户在筛选其他用户的时候，主要是用 age 和 sex 两个字段进行查询，而 nickname 和 description 两个字段主要用于展示，一般不会在业务查询中用到。description 本身又比较长，因此我们可以将这两个字段独立到另外 一张表中，这样在查询 age 和 sex 时，就能带来一定的性能提升。 

### 水平分表

水平分表适合表**行数特别大的表**，有的公司要求单表行数超过 5000 万就必须进行分表，这个数字可以作为参考，但并不是绝对标准，**关键还是要看表的访问性能**。对于一些比较复杂的表，可能超过 1000 万就要分表了；而对于一些简单的表，即使存储数据超过 1 亿行，也可以不分表。 但不管怎样，当看到表的数据量达到千万级别时，作为架构师就要警觉起来，因为这很可能是架构的性能瓶颈或者隐患。 

水平分表相比垂直分表，**会引入更多的复杂性**，例如**要求全局唯一的数据id该如何处理**  

## 全局唯一的id该如何处理 

### 主键自增

以最常见的用户 ID 为例，可以按照 1000000 的范围大小进行分段，1 ~ 999999 放到表 1中， 1000000 ~ 1999999 放到表2中，以此类推。

**复杂点**：**分段大小的选取**

* 分段太小会导致切分后子表数量过多，增加维护复杂度；
* 分段太大可能会导致单表依然存在性能问题，一般建议分段大小在 100 万至 2000 万之间，具体需要根据业务选取合适 的分段大小。 

**优点**：可以随着数据的增加**平滑地扩充新的表**。

* 例如，现在的用户是 100 万，如果增加到 1000 万， 只需要增加新的表就可以了，原有的数据不需要动。 

**缺点**：**分布不均匀**。

假如按照 1000 万来进行分表，有可能某个分段实际存储的数据量只有 1 条，而 另外一个分段实际存储的数据量有 1000 万条。

### 取模

同样以用户 ID 为例，假如我们一开始就规划了 10 个数据库表，可以简单地用 user_id % 10 的值来 表示数据所属的数据库表编号，ID 为 985 的用户放到编号为 5 的子表中，ID 为 10086 的用户放到编号 为 6 的子表中。 

**复杂点**：初始表数量的确定。表数量太多维护比较麻烦，表数量太少又可能导致单表性能存在问题。

**优点**：表分布比较均匀。 

**缺点**：扩充新的表很麻烦，所有数据都要重分布。  

### 雪花算法

雪花算法是由Twitter公布的**分布式主键生成算法**，它能够保证不同表的主键的不重复性，以及相同表的主键的有序性

> 核心思想： 长度共64bit（一个long型） 

首先是**一个符号位**，**1bit**标识，由于long基本类型在Java中是带符号的，最高位是符号位，正数是0，负数是1，所以id一般是正数，最高位是0。 

**41bit时间截(毫秒级)**，存储的是时间戳的差值（当前时间戳 - 开始时间戳)，结果约等于69.73年。 

**10bit作为机器的ID**（5个bit是数据中心，5个bit的机器ID，可以部署在1024个节点）

**12bit作为毫秒内的流水号**（意味着每个节点在每毫秒可以产生 4096 个 ID）

**优点：整体上按照时间自增排序，并且整个分布式系统内不会产生ID碰撞，并且效率较高。**

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/03-mybatis-plus-20230325-03.png)

# 条件构造器 Wrapper

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/03-mybatis-plus-20230325-05.png)

> Wrapper ： 条件构造抽象类，最顶端父类 

* AbstractWrapper ： **用于查询条件封装，生成 sql 的 where 条件** 
  * QueryWrapper ： 查询条件封装 
  * UpdateWrapper ： Update 条件封装 
  * AbstractLambdaWrapper ： 使用 Lambda 语法 
    * LambdaQueryWrapper ：用于 Lambda 语法使用的查询 Wrapper 
    * LambdaUpdateWrapper ： Lambda 更新封装 Wrapper

## QueryWrapper  

### 组装查询条件

**查询用户名包含a，年龄在20到30之间，并且邮箱不为null的用户信息**

```java
@Test
public void testQueryWrapper() {
    // 查询用户名包含a，年龄在20到30之间，并且邮箱不为null的用户信息
    /*
       SELECT id,username AS name,age.email,is_deleted
       FROM user
       WHERE is_deleted=0
           AND (name LIKE ?
           AND age BETWEEN ?
           AND ? AND
           email IS NOT NULL)
    */
    // 创建查询条件构造器对象
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    // 组装查询条件
    queryWrapper.like("name", "a").
            between("age", 20, 30)
            .isNotNull("email");

    // 查询
    List<User> users = userMapper.selectList(queryWrapper);

    // 输出
    users.forEach(System.out::println);
}
```

### 组装排序条件

**按年龄降序查询用户，如果年龄相同则按id升序排列**

```java
@Test
public void testOrderWrapper() {
    // 按年龄降序查询用户，如果年龄相同则按id升序排列
    /*
        SELECT id,username AS name,age,email,is_deleted
        FROM user
        WHERE is_deleted=0
        ORDER BY
            age DESC,
            id ASC
    */
    // 创建查询条件构造器对象
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    // 组装查询条件
    queryWrapper.orderByDesc("age").orderByAsc("id");

    // 查询
    List<User> users = userMapper.selectList(queryWrapper);

    // 输出
    users.forEach(System.out::println);

}
```

### 组装删除条件 

**删除email为空的用户**

```java
@Test
public void testDeleteWrapper() {
    // 删除email为空的用户
    // DELETE FROM user WHERE (email IS NULL)
    // 创建删除条件构造器对象
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    // 组装删除条件
    queryWrapper.isNull("email");

    int result = userMapper.delete(queryWrapper);
    System.out.println("受影响的行数：" + result);

}
```

### 设置查询字段 

**查询用户信息的username和age字段**

```java
@Test
public void testSelectSubColumn() {
    // 查询用户信息的username和age字段
    // SELECT name,age FROM user WHERE is_deleted=0
    // 创建条件构造器对象
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    // 设置查询字段
    queryWrapper.select("name", "age");

    List<Map<String, Object>> maps = userMapper.selectMaps(queryWrapper);
    maps.forEach(System.out::println);
}
```

### 字段 IN 的子查询

```java
@Test
public void testSelectInSql() {
    // 查询id小于等于3的用户信息

    /*
        SELECT *
        FROM user
        WHERE is_deleted=0 AND (id IN
        (
            SELECT id
            FROM user
            WHERE id <= 3
        ))
     *
     */

    // 创建条件构造器对象
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    // 组装查询条件
    queryWrapper.inSql("id", "SELECT id FROM user WHERE id <= 3");

    List<User> list = userMapper.selectList(queryWrapper);
    list.forEach(System.out::println);
}
```

## LambdaQueryWrapper 

```java
@Test
public void testLambdaQueryWrapper() {
    //定义查询条件，有可能为null（用户未输入）
    String name = "a";
    Integer ageBegin = 10;
    Integer ageEnd = 24;

    //  SELECT id,name,age,email,is_deleted FROM user WHERE is_deleted=0 AND (name LIKE ? AND age >= ? AND age <= ?)
    
    LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();

    queryWrapper.like(StringUtils.isNotBlank(name), User::getName, name)
            .ge(ageBegin != null, User::getAge, ageBegin)
            .le(ageEnd != null, User::getAge, ageEnd);
    List<User> users = userMapper.selectList(queryWrapper);
    users.forEach(System.out::println);
}
```

##  UpdateWrapper 

**将（年龄大于20或邮箱为null）并且用户名中包含有a的用户信息修改**

```java
@Test
public void testUpdateWrapper() {
    // 将（年龄大于20或邮箱为null）并且用户名中包含有a的用户信息修改
    // UPDATE user SET age=?,email=? WHERE is_deleted=0 AND (name LIKE ? AND (age > ? OR email IS NULL))
    // 创建更新条件构造器
    UpdateWrapper<User> updateWrapper = new UpdateWrapper<>();

    updateWrapper.set("age", 18)
            .set("email", "14725888@qq.com")
            .like("name", "a")
            .and(i->i.gt("age", 20).or().isNull("email"));

    int result = userMapper.update(null, updateWrapper);
    System.out.println(result);
}
```

## LambdaUpdateWrapper 

```java
@Test
public void testLambdaUpdateWrapper() {
    // 组装set子句
    LambdaUpdateWrapper<User> updateWrapper = new LambdaUpdateWrapper<>();
    
    // UPDATE user SET age=?,email=? WHERE is_deleted=0 AND (name LIKE ? AND (age < ? OR email IS NULL))
    updateWrapper
            .set(User::getAge, 18)
            .set(User::getEmail, "user@atguigu.com")
            .like(User::getName, "a")
            .and(i -> i.lt(User::getAge, 24).or().isNull(User::getEmail)); //lambda表达式内的逻辑优先运算
    User user = new User();
    int result = userMapper.update(user, updateWrapper);
    System.out.println("受影响的行数：" + result);
}
```

# 插件

## 分页插件

> MyBatis Plus 自带分页插件，只要简单的配置即可实现分页功能 

```java
@Configuration
public class MyBatisPlusConfig {

    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        
        return interceptor;
    }
}
```

```java
@Test
public void testPage(){

    // 设置分页参数
    Page<User> page = new Page<>(1, 5);
    userMapper.selectPage(page, null);
    // 获取分页数据
    List<User> list = page.getRecords();
    list.forEach(System.out::println);
    System.out.println("当前页："+page.getCurrent());
    System.out.println("每页显示的条数："+page.getSize());
    System.out.println("总记录数："+page.getTotal());
    System.out.println("总页数："+page.getPages());
    System.out.println("是否有上一页："+page.hasPrevious());
    System.out.println("是否有下一页："+page.hasNext());
}
```

## 自定义分页

> UserMapper 中定义接口方法 

```java
/**
 * 根据年龄查询用户列表，分页显示
 *
 * @param page 分页对象,xml中可以从里面进行取值,传递参数 Page 即自动分页,必须放在第一位
 * @param age 年龄
 * @return
 */
IPage<User> selectPageVo(@Param("page") Page<User> page, @Param("age") Integer age);
```

> mapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--DTD约束-->
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.cyan.mp.mapper.UserMapper">

    <!--SQL片段，记录基础字段-->
    <sql id="BaseColumns">id,`name`,age,email</sql>
    <!--IPage<User> selectPageVo(Page<User> page, Integer age);-->

    <select id="selectPageVo" resultType="com.cyan.mp.model.User">
        SELECT <include refid="BaseColumns"></include> FROM `user` WHERE age > #{age}
    </select>
</mapper>
```

> Test

````java
 @Test
    public void testSelectPageVo(){
        // 设置分页参数
        Page<User> page = new Page<>(1, 5);
        userMapper.selectPageVo(page, 18);
        
        // 获取分页数据
        List<User> list = page.getRecords();
        list.forEach(System.out::println);
        System.out.println("当前页："+page.getCurrent());
        System.out.println("每页显示的条数："+page.getSize());
        System.out.println("总记录数："+page.getTotal());
        System.out.println("总页数："+page.getPages());
        System.out.println("是否有上一页："+page.hasPrevious());
        System.out.println("是否有下一页："+page.hasNext());
    }
````



## 乐观锁插件

> 场景模拟

 一件商品，成本价是80元，售价是100元。老板先是通知小李，说你去把商品价格增加50元。小 李正在玩游戏，耽搁了一个小时。正好一个小时后，老板觉得商品价格增加到150元，价格太高，可能会影响销量。又通知小王，你把商品价格降低30元。 

此时，小李和小王同时操作商品后台系统。

* 小李操作的时候，系统先取出商品价格100元；

* 小王也在操作，取出的商品价格也是100元。小李将价格加了50元，并将100+50=150元存入了数据库；

* 小王将商品减了30元，并将100-30=70元存入了数据库。
* 是的，**如果没有锁，小李的操作就完全被小王的覆盖了**。 

现在商品价格是70元，比成本价低10元。几分钟后，这个商品很快出售了1千多件商品，老板亏1万多。  

上面的故事，如果是**乐观锁**，小王**保存价格前，会检查下价格是否被人修改过了**。如果被修改过 了，则**重新**取出的被修改后的价格，150元，这样他会将120元存入数据库。 

如果是**悲观锁**，小李取出数据后，小王只能**等小李操作完之后**，才能对价格进行操作，也会保证 最终的价格是120元。  

### 乐观锁

乐观锁（Optimistic Lock）： 每次去拿数据的时候都**认为别人不会修改**。所以不会上锁，但是如果想要更新数据，则会**在更新前检查在读取至更新这段时间别人有没有修改过这个数据**。如果修改过，则**重新读取，再次尝试更新**，循环上述步骤直到更新成功（当然也允许更新失败的线程放弃操作），乐观锁**适用于多读**的应用类型，这样可以提高吞吐量

相对于悲观锁，在对数据库进行处理的时候，乐观锁并不会使用数据库提供的锁机制。

一般的**实现乐观锁的方式**就是**记录数据版本**或者是**时间戳**来实现，不过使用版本记录是最常用的。 

### 悲观锁

悲观锁（Pessimistic Lock）：每次去拿数据的时候**都认为别人会修改**。所以每次**在拿数据的时候都会上锁**。这样别人想拿数据就被挡住，直到悲观锁被释放，悲观锁中的共享资源**每次只给一个线程使用**，其它线程阻塞，用完后再把资源转让给其它线程

但是在效率方面，处理加锁的机制会**产生额外的开销**，还有**增加产生死锁的机会**。另外还会降低并行性，如果已经锁定了一个线程A，其他线程就必须等待该线程A处理完才可以处理

数据库中的`行锁，表锁，读锁（共享锁），写锁（排他锁），以及syncronized实现的锁`均为**悲观锁**

### 模拟冲突

> **数据库中增加商品表** 

```sql
CREATE TABLE t_product
(
    id BIGINT(20) NOT NULL COMMENT '主键ID',
    NAME VARCHAR(30) NULL DEFAULT NULL COMMENT '商品名称',
    price INT(11) DEFAULT 0 COMMENT '价格',
    VERSION INT(11) DEFAULT 0 COMMENT '乐观锁版本号',
    PRIMARY KEY (id)
);
```

> **后端代码**

```java
@Data
public class Product {
    private Long id;
    private String name;
    private Integer price;
    private Integer version;
}
```

```java
public interface ProductMapper extends BaseMapper<Product> {
}
```

> **测试**

```java
@Test
public void testConcurrentUpdate() {
    // 1、小李
    Product p1 = productMapper.selectById(1L);
    System.out.println("小李取出的价格：" + p1.getPrice());
    // 2、小王
    Product p2 = productMapper.selectById(1L);
    System.out.println("小王取出的价格：" + p2.getPrice());
    // 3、小李将价格加了50元，存入了数据库
    p1.setPrice(p1.getPrice() + 50);
    int result1 = productMapper.updateById(p1);
    System.out.println("小李修改结果：" + result1);
    // 4、小王将商品减了30元，存入了数据库
    p2.setPrice(p2.getPrice() - 30);
    int result2 = productMapper.updateById(p2);
    System.out.println("小王修改结果：" + result2);
    // 最后的结果
    Product p3 = productMapper.selectById(1L);
    //价格覆盖，最后的结果：70
    System.out.println("最后的结果：" + p3.getPrice());
}
```

### 乐观锁实现

 数据库中添加**version**字段，取出记录时，获取当前**version** 

```sql
SELECT id,`name`,price,`version` FROM product WHERE id=1
```

**更新时，version + 1，如果where语句中的version版本不对，则更新失败** 

```sql
UPDATE product SET price=price+50, `version`=`version` + 1 WHERE id=1 AND`version`=1
```

> 在实体类的字段上加上`@Version`注解

```JAVA
@Data
public class Product {
    private Long id;
    private String name;
    private Integer price;
    @Version
    private Integer version;
}
```

> 配置插件添加乐观锁

**spring.xml方式**

```xml
bean class="com.baomidou.mybatisplus.extension.plugins.inner.OptimisticLockerInnerInterceptor" id="optimisticLockerInnerInterceptor"/>

<bean id="mybatisPlusInterceptor" class="com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor">
    <property name="interceptors">
        <list>
            <ref bean="optimisticLockerInnerInterceptor"/>
        </list>
    </property>
</bean>
```

**SpringBoot注解方式**

```java
@Bean
public MybatisPlusInterceptor mybatisPlusInterceptor() {
    MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
    interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
    return interceptor;
}
```

> **测试修改冲突**  

 小李查询商品信息： 

```sql
SELECT id,name,price,version FROM t_product WHERE id=? 
```

小王查询商品信息：

```sql
SELECT id,name,price,version FROM t_product WHERE id=? 
```

小李修改商品价格，自动将version+1 

```sql
UPDATE t_product SET name=?, price=?, version=? WHERE id=? AND version=? 
```

Parameters: 外星人笔记本(String), 150(Integer), 1(Integer), 1(Long), 0(Integer) 

小王修改商品价格，此时version已更新，条件不成立，修改失败 

```sql
UPDATE t_product SET name=?, price=?, version=? WHERE id=? AND version=? 
```

Parameters: 外星人笔记本(String), 70(Integer), 1(Integer), 1(Long), 0(Integer)

 最终，小王修改失败，查询价格：150 

```sql
SELECT id,name,price,version FROM t_product WHERE id=? 
```

**优化流程**

```java
@Test
public void testConcurrentVersionUpdate() {
    // 小李取数据
    Product p1 = productMapper.selectById(1L);
    // 小王取数据
    Product p2 = productMapper.selectById(1L);
    // 小李修改 + 50
    p1.setPrice(p1.getPrice() + 50);
    int result1 = productMapper.updateById(p1);
    System.out.println("小李修改的结果：" + result1);
    // 小王修改 - 30
    p2.setPrice(p2.getPrice() - 30);
    int result2 = productMapper.updateById(p2);
    System.out.println("小王修改的结果：" + result2);
    if(result2 == 0){
    // 失败重试，重新获取version并更新
    p2 = productMapper.selectById(1L);
        p2.setPrice(p2.getPrice() - 30);
    result2 = productMapper.updateById(p2);
    }
    System.out.println("小王修改重试的结果：" + result2);
    // 老板看价格
    Product p3 = productMapper.selectById(1L);
    System.out.println("老板看价格：" + p3.getPrice());
}
```

# 通用枚举

## 创建通用枚举类型

```java
@Getter
public enum SexEnum {
    
    MALE(1, "男"),
    FEMALE(2, "女");
    
    @EnumValue
    private Integer sex;
    private String sexName;
    
    SexEnum(Integer sex, String sexName) {
        this.sex = sex;
        this.sexName = sexName;
    }
}
```

## 配置扫描通用枚举

```yml
mybatis-plus:
  configuration:
    # 配置MyBatis日志
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  global-config:
    db-config:
      # 配置MyBatis-Plus操作表的默认前缀
      table-prefix: t_
      # 配置MyBatis-Plus的主键策略
      id-type: auto
  # 配置扫描通用枚举
  type-enums-package: com.cyan.mp.enums
```

## 运行测试

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {

    private Long id;
    private String name;
    private Integer age;
    private String email;
    @TableLogic
    private Integer isDeleted;
    private SexEnum sex;
}
```

> 运行测试

```java
@Test
public void testSexEnum(){
    User user = new User();
    user.setName("Enum");
    user.setAge(20);
    // 设置性别信息为枚举项，会将@EnumValue注解所标识的属性值存储到数据库
    user.setSex(SexEnum.MALE);
    // INSERT INTO t_user ( username, age, sex ) VALUES ( ?, ?, ? )
    // Parameters: Enum(String), 20(Integer), 1(Integer)
    userMapper.insert(user);
}
```

# 官方教程

[代生成器 阅读请点我 ! ](https://baomidou.com/pages/779a6e/#%E4%BD%BF%E7%94%A8)

[多数据源 阅读请点我 !](https://baomidou.com/pages/a61e1b/#mybatis-mate)