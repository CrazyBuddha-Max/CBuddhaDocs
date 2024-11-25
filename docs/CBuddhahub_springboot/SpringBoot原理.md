（原创）springboot原理
***
## 一、配置优先级
##### > 命令行参数 > java系统属性 > properties文件 > yml文件(主流) > 环境变量 

> 1.java系统属性和命令行参数设置在IDEA中的实现如图：上面的箭头是java系统属性配置端口号为9091，下面的箭头是命令行参数配置端口号为10010。
> ![img](https://img2023.cnblogs.com/blog/3467365/202408/3467365-20240807004639507-1183031855.png)

> 2.执行java指令，运行jar包时设置命令行参数和java系统属性的端口号：
> java -jar -Dserver.port=9091 springboot-vue-0.0.1-SNAPSHOT.jar --server.port=10010

## 二、Bean管理
##### 1.获取bean
- 默认情况下，Spring项目启动时，会把bean都创建好放在IOC容器中，如果想主动获取这些bean，可以通过以下方式：
- 1.根据name获取bean：
```java
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
Object bean = context.getBean("beanName");
```
- 2.根据类型获取bean：
```java
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
Object bean = context.getBean(User.class);
```
- 3.根据name和类型获取bean：
```java
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
Object bean = context.getBean("beanName", User.class);
```
##### 2.配置作用域
> 1.默认情况下，Spring创建的Bean都是单例的，即在整个Spring容器中，Bean只有一个实例。
> 2.在Spring中，可以通过scope属性来配置Bean的作用域，常用的作用域有：
> - singleton：单例作用域，默认值，在整个Spring容器中，Bean只有一个实例。
> - prototype：原型作用域，每次使用Bean时，都会创建一个新的实例（非单例）。
> - request：请求作用域，每次HTTP请求范围内都会创建一个新的实例。
> - session：会话作用域，每次HTTP会话范围内都会创建一个新的实例。
> - application：应用作用域，每个Spring应用范围内都会创建一个实例。

##### 3.配置作用域的示例：
```java
@Configuration
public class AppConfig {
    @Bean
    @Scope("prototype") //每次使用bean时都会创建一个新的实例
    @Lazy //默认singleton的bean在容器启动时被创建。当有该注解时延迟初始化，即第一次使用该bean时才会创建。
    
    public User user() {
        return new User();
    }
}
```
##### 4.第三方bean的管理
加上注解@Configuration配置类，加上注解@Bean配置第三方bean
```java
@Configuration
public class AppConfig {
    @Bean
    public User user() {
        return new User();
    }
}
```
对第三方bean进行依赖注入：
> 如果第三方bean需要依赖其他bean，直接在bean定义方法中设置形参即可，容器会根据类型自动装配。
对bean的名称修改：
> 通过@Bean注解的name/value属性可以修改bean的名称，如果不设置name属性，默认使用方法名作为bean的名称。
## 三、Springboot原理
> 问:Springboot比spring多了什么？答:多了起步依赖和自动配置。
### 1.起步依赖
#### 定义
> 起步依赖就是项目启动时就将常用的依赖配置好，简化了依赖配置。
#### 原理
> 利用maven传递依赖的特性.
### 2.自动配置（springboot原理的主要内容）
#### 定义
> 当spring容器启动后，一些配置类、bean对象自动存入IOC容器中，不需要我们手动配置。从而简化了开发，省去了繁琐的配置操作。
#### 自动配置的实现方式
> 方案一：通过@ComponentScan注解扫描指定包下的类，将类中的bean存入IOC容器中。
```java
@Configuration
@ComponentScan("com.example.demo") <---指定扫描的包
public class AppConfig {
}
```
> 方案二：通过@Import导入。使用@Import导入的类会被加载到IOC容器中，导入的形式有以下几种：
> - 1.导入普通类：
> - 2.导入配置类：
> - 3.导入ImportSelector接口的实现类：
> - 4.EnableXxxx注解，封装@Import注解：
```java
@Import({TokenParser.class}) //导入普通类，交给ioc容器管理
@Import({HeaderConfig.class}) //导入配置类，交给ioc容器管理
@Import({MyImportSelector.class}) //导入ImportSelector接口实现类，交给ioc容器管理

@EnableHeaderConfig //导入EnableHeaderConfig注解，EnableHeaderConfig注解中封装了@Import注解，导入HeaderConfig配置类，交给ioc容器管理
@SpringBootApplication

public class SpringbootWebConfig2Application {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootWebConfig2Application.class, args);
    }

}
```
#### 自动配置的底层原理
##### 源码追踪: 
###### @SpringBootApplication注解标识在SpringBoot工程引导类上，是最重要的注解。其中包含了三个子注解：@EnableAutoConfiguration：@ComponentScan、@SpringBootConfiguration。
> 1.@SpringBootConfiguration：标识该类是一个配置类，相当于@Configuration注解。

> 2.@ComponentScan：标识该类是一个组件扫描器，默认扫描当前引导类所在的包及其子包。

> 3.@EnableAutoConfiguration：开启自动配置，是SpringBoot自动配置的核心注解。其中包含了@Import注解，导入AutoConfigurationImportSelector类，该类实现了ImportSelector接口，重写了selectImports方法，该方法返回了一个字符串数组，数组中的每个字符串都是一个配置类的全限定名，这些配置类会被Spring容器加载，从而实现自动配置。

###### @conditional注解
- 作用
- > 按照一定的条件进行判断，在满足给定条件后才会注册对应的bean对象到Spring IOC容器中。
- 位置
- > 方法、类。
- 本身是一个父注解，派生出很多子注解：
> - @ConditionalOnProperty：当配置文件中有对应的属性和值时，才创建该bean到IOC容器。

> - @ConditionalOnMissingBean：当容器中不存在指定类型或者名称的bean时，才创建该bean到IOC容器。

> - @ConditionalOnClass：当类路径下存在指定类型的类或者字节码文件时，才创建该bean到IOC容器。

###### 自定义starter
- 定义
- > 自定义starter，就是自定义一个starter工程，将常用的依赖配置好，并编写自动配置类，将自动配置类封装到@EnableXxxx注解中，将@EnableXxxx注解封装到@SpringBootApplication注解中，从而实现自动配置。

- 实现步骤

- 案例：自定义阿里云oss-starter，完成阿里云OSS操作工具类AliyunossUtil的自动配置。引入依赖之后，要想使用阿里云OSS，注入AliyunOSSUtils直接使用即可。
- 实现方法：
>1.创建aliyun-oss-spring-boot-starter模块。

>2.创建aliyun-oss-spring-boot-autoconfigure模块，在starter中引入该模块。

>3.在aliyun-oss-spring-boot-autoconfigure模块中定义自动配置功能，并定义自动配置文件META-INF/spring/xxxx.imports。
- 详细见d:/code/ProgramStar/springbootProg/springboot-autoconfiguration-test
小知识：
> mvn idea:module 是一个 Maven 命令，用于在 IntelliJ IDEA 中生成或更新 Maven 项目的模块文件。


### web后端开发总结：
1.三层架构：控制器层controller（接受请求响应数据）、业务层service（具体的业务逻辑处理）、数据访问层/持久层dao（处理数据访问操作，例如crud）。
2.对于通用的业务处理，例如统一的字符编码。可以借助javaweb三大组件之一的过滤器filter、或者用spring提供的拦截器interceptor来实现。
3.为了实现三层架构层与层之间的解耦，可以使用spring框架当中的第一大核心IOC控制反转与DI依赖注入来实现。
4.掌握aop面向切面编程，spring中的事务管理，全局异常处理以及传统会话技术session、cookie和jwt令牌，阿里云OSS对象存储服务以及通过mybatis持久层框架对数据库进行操作等。
5.springMVC是spring framework框架中的一部分，用于实现web后端开发，用来简化原始的servlet开发，实现mvc架构。包含接受请求响应数据拦截器和全局异常处理的功能。spring framework包含IOC控制反转与DI依赖注入、AOP编程和事务管理的功能。SpringMVC、Spring framework、Mybatis称为SSM框架。基于传统的SSM框架开发的步骤比较繁琐、效率低下，在现在企业项目开发中，一般使用springboot框架来简化开发，springboot框架是spring框架的升级版，基于spring框架开发，简化了spring框架的开发，提供了自动配置的功能，使用springboot框架开发，可以大大提高开发效率。