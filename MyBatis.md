# 什么是MyBatis?

MyBatis是一款优秀的持久层的框架, 支持定制化的SQL, 存储过程以及高级映射.

#### 安装

将mybatis-x.x.x.jar文件置于classpath中即可

在maven构建的项目中, 在pom.xml中加入dependency代码


```xml
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>x.x.x</version>
</dependency>
```

#### 从xml中构建SqlSessionFactory

每个基于MyBatis的应用都是以一个SqlSessionFactory 的实例中心的.

SqlSessionFactory通过SqlSessionFactoryBuilder获得.

SqlSessionFactoryBuilder通过xml配置文件或者Configuration的实例构建出SqlSessionFactory

```java
String resource = "org/mybatis/example/mybatis-config.xml"
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

XML配置文件中包含了对MyBatis系统的核心设置: 数据源, 事务作用域, 事务管理器

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
</configuration>
```

environment元素体中包含了事务管理和连接池的配置.

mappers元素包含一组mapper映射器

#### 不使用xml构建SqlSessionFacory

```java
DataSource dataSource = BlogDataSourceFactory.getBlogDataSource();
TransactionFactory transactionFactory = new JdbcTransactionFactory();
Environment environment = new Environment("development", transactionFactory, dataSource);
Configuration configuration = new Configuration(environment);
configuration.addMapper(BlogMapper.class);
SqlSessionFacory sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration)

```

#### 从SqlSessionFactory中获取SqlSession

SqlSession实例来直接这行已映射的Sql语句


##### 旧式
```java
  SqlSession session = sqlSessionFactory.openSession();
  try {
    Blog blog = (Blog) session.selectOne("org.mybatis.example.BlogMapper.selectBlog", 101);
  } finally {
    session.close();
  }
```

##### 新式

```java
  SqlSession session = SqlSessionFactory.openSession();
  try{
      BlogMapper mapper = session.getMapper(BlogMapper.class);
      Blog blog = mapper.selectBlog(101);
  } finally {
      sesssion.close();
  }
```

#### 已经映射的SQL语句

通过xml定义语句

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.mybatis.example.BlogMapper">
  <select id="selectBlog" resultType="Blog">
    select * from Blog where id = #{id}
  </select>
</mapper>
```
通过Java注解来映射语句

```java
public interface BlogMapper{
    @Select("Select * FROM blog WHERE id = #{id}")
    Blog selectBlog(int id);
}
```

Java 注解对于稍微复杂的语句就会使得代码更加混乱, 当需要做复杂操作, 最好使用xml来定义


#### 作用域(Scope)和生命周期

> 依赖注入框架可以创建线程安全的, 基于事务的SqlSession和映射器(mapper)并将它们直接注入到你的bean中
> 因此可以直接忽略它们的生命周期.


###### SqlSessionFactoryBuilder

这个类可以被实例化,使用和丢弃, 一旦创建SqlSessionFactory, 就不在需要了, 最佳作用域是方法作用域(局部方法变量).

###### SqlSessionFactory

SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在, 不应该对它进行清除和重建
最佳作用域是应用作用域, 使用单例模式或静态单例模式

###### SqlSession

每个线程都应该有它自己的SsqlSession实例. SqlSession的实例不是线程安全的, 因此不能被共享
最佳作用域是请求或者方法作用域, 绝对不能将SqlSession的实例的引用放在一个静态域中, 类的实例变量也不行

每次收到HTTP请求, 就可以打开一个SqlSession, 返回一个响应, 就关闭它
关闭很重要, 放在finally块中, 确保每次都能执行关闭

```java
SqlSession session = sqlSessionFactory.openSession();
try {
  // do work
} finally {
  session.close();
}

```

###### 映射实例 (Mapper Instances)

映射器是创建用来绑定映射语句的接口.
映射器接口实例是从SqlSession中获取的.

映射器实例最佳作用域是方法域.
不需要显示的关闭映射器实例

```java
SqlSession session = sqlSessionFactory.openSession();
try {
  BlogMapper mapper = session.getMapper(BlogMapper.class);
  // do work
} finally {
  session.close();
}
```

#### XML 映射配置文件

MyBatis 的配置文件包含了会深深影响MyBatis行为的设置(settings)和属性(properties)信息

- configuration 配置
    * properites 属性
    * settings 设置
    * typeAliases 类型别名
    * typeHandlers 类型处理器
    * objectFactory 对象工厂
    * plugins 插件
    * environments 环境
        * environment 环境变量
            * transactionManager 事务管理器
            * dataSource 数据源
    * databaseIdProvider 数据库厂商标示
    * mapper 映射器


###### properties
