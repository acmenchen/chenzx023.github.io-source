---
title: springboot项目整合mybatis
date: 2020-05-28 15:51:57
tags: [java,springboot,框架]
categories: [java]
---
今天这篇我们将具体说说如何在<font color=#e96900>`Spring Boot`</font>中整合<font color=#e96900>`MyBatis`</font>完成关系型数据库的增删改查操作。

学习<font color=#e96900>`Mybatis`</font>我们先来了解一下什么<font color=#e96900>`ORM`</font>，<font color=#e96900>`ORM`</font>即<font color=#e96900>`Object-Relationl Mapping`</font>，它的作用是在关系型数据库和对象之间作一个映射，这样，我们在具体的操作数据库的时候，就不需要再去和复杂的<font color=#e96900>`SQL`</font>语句打交道，只要像平时操作对象一样操作它就可以了 。

下面我介绍一款在国内使用量比较多的<font color=#e96900>`ORM`</font>框架，它就是<font color=#e96900>`Mybatis`</font>。
## 整合mybatis

##### 1.  创建好一个springboot项目后，首先在pom.xml配置文件引入MyBatis以及MySQL Connector


```xml
<dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
  </dependency>
<dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.2</version>
</dependency>
```

关于<font color=#e96900>`mybatis-spring-boot-starter`</font>的版本需要注意：

- <font color=#e96900>`2.1.x`</font>版本适用于：Mybatis3.5+、Java8+、Spring Boot 2.1+

- <font color=#e96900>`2.0.x`</font>版本适用于：Mybatis3.5+、Java8+、Spring Boot 2.0/2.1

- <font color=#e96900>`1.3.x`</font>版本适用于：Mybatis3.4+、Java6+、Spring Boot1.5

其中，目前还在维护的版本是2.1.x和1.3.x


##### 2. 在resources目录下创建 <font color=#e96900>`application.properties`</font>文件

```yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver  #mysql驱动
    url: jdbc:mysql://127.0.0.1:3306/test  #url
    username: root # 数据库用户名
    password: 123456 #数据库密码
```

##### 3. Mysql 中创建一张用来测试的表，比如： User表。
      具体创建命令如下：

```mysql
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(50) DEFAULT NULL,
  `age` varchar(50) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8;
```

##### 4. 创建User表的映射对象User

```java
   public class User {
       private Integer id;
   
       private String name;
   
       private int age;
   	
       ...这里省略get、set方法
   }
```

##### 5.  创建User表的操作接口：UserMapper。在接口中定义两个数据操作，一个插入。一个查询，用于后续验证。

 ```java
 @Mapper
public interface UserMapper {
    // 查询所有用户
    @Select({"select * from user"})
    List<User> selectUserList();
    // 插入一条用户信息
    @Insert({"insert into user (name, age) values (#{name}, #{age})"})
    int User(@Param("name") String name, @Param("age") int age);
}
 ```

##### 6. 创建User表的操作服务接口: UserService。

   > 好处：封装Service层的业务逻辑有利于业务逻辑的独立性和重复利用性。

```java
public interface UserService {
    // 查询所有用户
    List<User> selectUserList();
    // 创建用户
    int createUser(String name, int age);
    
}
```

##### 7. 创建UserService接口的实现类

```java
@Service
public class UserServiceImpl implements UserService {
    @Autowired(required = false)
    private UserMapper usermapper;
	@Override
	public List<User> selectUserList() {
		return usermapper.selectUserList();
	}
	@Override
	public int createUser(String name, int age) {
		return usermapper.createUser(name, age);
	}
    
}
```

##### 8. 创建控制器。具体如下

```java
@RestController
public class HomeController {
    
    @Autowired
    private UserServiceImpl userService;
    @RequestMapping("/users")
    public Result users() {
        List<User> userList = userService.selectUserList();
        return Result.success(userList);
    }
    @RequestMapping("createUser")
    public Result createUser(@RequestParam("name") String name, @RequestParam("age") int age){
        System.out.println(name+":"+age);
        int result = userService.createUser(name, age);
        return Result.success(result);
    }

}
```

- Result 可自己封装一个结果统一返回工具类

代码具体如下：

```java
public class Result {
    // 错误码
    private String  code;
    // 错误返回信息
    private String msg;
    // 返回结果集
    private Object data  = null;
 
    ...这里省略get、set方法
        
    public static Result success(Object object) {
        Result result = new Result();
        result.setCode("0");
        result.setMsg("操作成功");
        result.setData(object);
        return result;
    }  

    public static Result success(String msg, Object data) {
        Result result = new Result();
        result.setCode("0");
        result.setMsg(msg);
        result.setData(data);
        return result;
    }    
    public static  Result error(String msg, Object data) {
        Result result = new Result();
        result.setCode("-1");
        result.setMsg(msg);
        result.setData(data);
        return result;
    }    
  }
```



## 注解配置说明

####  使用@Param

在之前的整合实例中我们已经使用了这种简单 的传参方式，如下：

```java
 @Insert({"insert into user (name, age) values (#{name}, #{age})"})
  int createUser(@Param("name") String name, @Param("age") int age);
```

这种方式很好理解，<font color=#e96900>`@Param` </font>中定义的<font color=#e96900>name</font>对应了Sql中的<font color=#e96900>#{name}</font>，<font color=#e96900>age</font>对应了Sql中的<font color=#e96900>#{age}</font>
