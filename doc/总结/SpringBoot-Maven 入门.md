# 			SpringBoot-Maven 入门

## 一.SpringBoot介绍

## 二.SpringBoot-Maven入门案例

### 步骤一：新建Maven Project

![1563520341694](C:\Users\zhangxiaoliang\AppData\Roaming\Typora\typora-user-images\1563520341694.png)

![1563520397173](C:\Users\zhangxiaoliang\AppData\Roaming\Typora\typora-user-images\1563520397173.png)





### 步骤二：pom文件添加依赖

```xml
 <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.0.RELEASE</version>
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
```



### 步骤三：编写启动类

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        // TODO Auto-generated method stub
        SpringApplication.run(Application.class, args);
    }

}
```

### 步骤四：修改端口 

新建application.properties文件 在其内写入server.port = 8089(端口号)

![1563520673070](C:\Users\zhangxiaoliang\AppData\Roaming\Typora\typora-user-images\1563520673070.png)

### 步骤五：启动

 run as Java Application

![1563520817343](C:\Users\zhangxiaoliang\AppData\Roaming\Typora\typora-user-images\1563520817343.png)

## 注意事项