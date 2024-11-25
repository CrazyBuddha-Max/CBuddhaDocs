# <span style='color:blue'>(原创) springboot实现分层解耦的方案</span> :smile:
***
###  1. 分层解耦的好处
- ==解决代码复用性差，难以维护的缺点。==
- ==提高代码的可读性，便于维护。==
- ==提高代码的扩展性，便于扩展。==
***
###  2. 分层解耦的实现之三层架构分层
- 控制层controller：==负责与前端交互，接收前端的请求，调用业务层处理请求，将处理结果返回给前端。==
- 业务逻辑层service：==负责处理业务逻辑，调用持久层获取数据。==
- 数据访问层(持久层)Dao：==负责数据库访问操作，包括数据的增删改查操作==

***
###  3. 分层解耦的实现之三层架构解耦
- ##### <span style='color:red'>定义理解</span>
    - <span style='color:orange'>内聚：</span>
      - <span style='color:green'>软件中各个功能模块内部的功能联系。</span>
    - <span style='color:orange'>耦合：</span>
      - <span style='color:green'>衡量软件中各个模块之间的依赖、关联的程度。</span>
    - <span style='color:orange'>控制反转：</span>
      - <span style='color:green'>(Inversion Of Control & IOC) 对象的创建控制权由程序自身转移到外部(容器)，这种思想叫做控制反转。</span>
    - <span style='color:orange'>依赖注入：</span>
      - <span style='color:green'>(Dependency Injection & DI) 容器为应用程序提供运行时，所依赖的资源，叫做依赖注入。</span>
    - <span style='color:orange'>Bean对象：</span>
      - <span style='color:green'>IOC容器中创建、管理的对象，叫做bean。</span>
> <span style='color:pink'>个人对IOC和DI的几点理解：</span>
> - <span style='color:orange'>在三层架构中，为了降低代码各个模块的依赖关联度，将各层中对象的创建、管理、依赖注入等操作都交给IOC容器来完成，各层之间通过IOC容器提供的接口来获取所需要的对象，从而实现各层之间的解耦。
> - <span style='color:orange'>IOC容器相当于房地产中介，买房者和卖房者之间并没有直接联系，而是通过中介来完成交易。中介的作用就是将卖方和买方联系起来，买方只需要告诉中介自己需要什么房子，中介就会将符合条件的房子推荐给买房者。</span>

- ##### <span style='color:blue'>解耦的具体实现</span>
  1. 把需要交给IOC容器处理的实现类的上方添加@Component注解，将实现类交给IOC容器管理。
  2. 在需要使用实现类的类上方添加@Autowired注解，将实现类注入到需要使用实现类的类中。
> <span style='color:pink'>个人理解：</span>
> - <span style='color:orange'>实现类中只需要声明对象，创建对象的工作交给IOC容器来完成。</span>
> - <span style='color:orange'>同一层架构的注解下的实现类只能有一个，如果有多个实现类需要注释掉其他实现类上的注解</span>

***
###  4. IOC控制反转详解
- ##### <span style='color:blue'>bean对象的声明</span>
    - 实际开发中使用Component衍生注解
      - <span style='color:red'>@Controller</span>-<span style='color:orange'>标注在控制器类上</span>
      - <span style='color:red'>@Service</span>-<span style='color:orange'>标注在业务类上</span>
      - <span style='color:red'>@Repository</span>-<span style='color:orange'>标注在数据访问类上(现在用@Mapper替代了))</span>
<br>
    > <span style='color:red'>注意：</span>
    > <span style='color:orange'>1. 声明bean时，可通过value属性指定bean的id，不指定时，默认id为当前类名首字母小写。</span>
    > <span style='color:orange'>2. spring集成web开发中，声明控制器bean只能用@Controller。</span>
- ##### <span style='color:blue'>bean组件扫描</span>
    - 默认扫描的包为当前包及其子包。
      - 方法1：可以通过@ComponentScan注解的value属性指定需要扫描的包。
      - 方法2：可以通过@ComponentScan注解的basePackages属性指定需要扫描的包。
    - @SpringBootApplication注解具有包扫描作用，默认扫描当前包机及其子包。

***
### 5. DI依赖注入详解
- ##### <span style='color:blue'>bean注入</span>
  - @Autowired注解，默认按照类型进行注入，如果存在多个相同类型的bean，则报错。解决方法如下：
    - > <span style='color:green'>方法一：</span>在控制层的@Autowired注解下加上@Qualifier("bean的id")注解，表示当前bean生效。![img](https://img2023.cnblogs.com/blog/3467365/202406/3467365-20240625141014691-543689726.png)
    - > <span style='color:green'>方法二：</span>在服务层@Service注解上加上@Primary注解，表示当前bean生效。![img](https://img2023.cnblogs.com/blog/3467365/202406/3467365-20240625141032535-1130257077.png)
    - > <span style='color:green'>方法三：</span>在控制层上删除@Autowired注解，再加上@Resource(name= "bean的id")注解，表示当前bean生效。![img](https://img2023.cnblogs.com/blog/3467365/202406/3467365-20240625141750384-369258935.png)
 
> 总结：@Resource注解和@Autowired注解都可以实现依赖注入，它们的区别是：
> - @Autowired是spring框架提供的，@Resource注解是JDK提供的。
> - @Autowired默认按照类型注入，@Resource注解默认按照名称注入。
***
