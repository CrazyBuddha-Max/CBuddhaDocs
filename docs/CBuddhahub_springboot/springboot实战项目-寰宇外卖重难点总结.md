# 思考
#### 前端和后端的请求地址不同，前端发送的请求，是如何请求到后端服务的？
> 可以通过nginx反向代理将前端发送的动态请求由nginx转发到后端服务。nginx其他优点：1.提高访问速度。2.进行负载均衡。3.安全性高，保护后端服务安全。

#### nginx负载均衡策略：
**1.轮询（默认）：按时间顺序依次将请求分发到不同的后端服务器。
2.weight：权重，默认为1，权重越高，分配的请求越多。
3.ip_hash：每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器。
4.fair：按后端服务器的响应时间来分配请求，响应时间短的优先分配。
5.url_hash：按访问url的hash结果来分配请求，使相同的url定向到同一个后端服务器。
6.least_conn：最少连接数，哪个机器连接数少就分发到哪个机器。
7.自定义：自定义负载均衡策略。**


#### Swagger接口文档生成工具
**Knife4j是为了java MVC框架集成Swagger生成API文档的工具，前身是swagger-bootstrap-ui。**
> 使用方式：1.导入Knife4j的maven坐标。2.在配置类中加入knife4j的相关配置。3.设置静态资源映射，为了接口文档页面的正常访问。
> 设置静态资源映射的原因：Swagger的页面是通过静态资源访问的，而SpringBoot默认的静态资源路径是“classpath:/static/”和“classpath:/public/”，所以需要设置静态资源映射。如果不设置静态资源映射，那么就会被当作动态请求，就会报404错误。


#### Swagger接口文档生成工具和YApi的区别
**1.Swagger是后端生成接口文档的工具，而YApi是前后端都可以生成接口文档的工具。**
**2.Swagger是开发阶段使用的框架，帮助后端开发人员做接口测试。而YApi是设计阶段使用的工具，目的是管理和维护接口。**

#### Knife4j的常见注解
**1.@Api：用在类上，表示对类的说明。value：表示说明的内容，name：接口名称。**
**2.@ApiOperation：用在方法上，表示对方法的说明。value：表示说明的内容，notes：备注。**
**3.@ApiModelProperty：用在属性上，表述属性信息**
**4.@ApiModel：用在类上，表示对类的说明。value：表示说明的内容，description：描述。**
> 作用是让接口文档的生成更加直观，方便前后端开发人员理解接口的用途和参数。


#### 具体开发
1.新增员工：
> 1.1. 根据新增员工接口设计DTO，因为当前端提交的数据和实体类对应的属性差别过大时，建议使用DTO封装数据。  
 
2.分页查询：
> 2.1. 使用PageHelper插件实现分页查询。
> PageHelper.startPage是一个常用的MyBatis分页插件，用于在执行查询时自动处理分页逻辑。其基本实现原理如下：
2.1.1. 拦截器机制：PageHelper通过MyBatis的拦截器机制，在查询方法被调用前进行拦截。它会在执行查询之前，读取传入的分页参数（如页码和每页大小）。
2.1.2. 保存分页参数：在拦截器中，PageHelper会将分页参数保存到一个线程局部变量中，这样每个线程可以独立地存储自己的分页信息。
2.1.3. 修改SQL语句：在拦截查询语句时，PageHelper会对SQL进行修改，添加LIMIT和OFFSET子句，从而实现分页效果。具体来说，它会根据当前页码和每页大小计算出要跳过的记录数（OFFSET）和要获取的记录数（LIMIT）。
2.1.4. 执行查询：修改后的SQL被传递到数据库执行，返回相应的分页结果。
2.1.5. 返回结果：最终，PageHelper将结果封装成分页对象（如PageInfo），提供分页相关的信息（如总记录数、总页数等），方便调用者使用。

> 2.2. 分页查询的数据中时间格式化，让前端显示正确的日期格式：
> 2.2.1. 使用@JsonFormat注解，指定日期格式。(只能单次生效，下次碰到同类型的还要加上该注解)
> 2.2.2. 在WebMvcConfig配置类中，添加日期格式化的配置。(一劳永逸，所有日期格式化都生效)
#### 具体实现代码：
```java
/**
 * 对象映射器:基于jackson将Java对象转为json，或者将json转为Java对象
 * 将JSON解析为Java对象的过程称为 [从JSON反序列化Java对象]
 * 从Java对象生成JSON的过程称为 [序列化Java对象到JSON]
 */
public class JacksonObjectMapper extends ObjectMapper {

    public static final String DEFAULT_DATE_FORMAT = "yyyy-MM-dd";
    //public static final String DEFAULT_DATE_TIME_FORMAT = "yyyy-MM-dd HH:mm:ss";
    public static final String DEFAULT_DATE_TIME_FORMAT = "yyyy-MM-dd HH:mm";
    public static final String DEFAULT_TIME_FORMAT = "HH:mm:ss";

    public JacksonObjectMapper() {
        super();
        //收到未知属性时不报异常
        this.configure(FAIL_ON_UNKNOWN_PROPERTIES, false);

        //反序列化时，属性不存在的兼容处理
        this.getDeserializationConfig().withoutFeatures(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);

        SimpleModule simpleModule = new SimpleModule()
                .addDeserializer(LocalDateTime.class, new LocalDateTimeDeserializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_TIME_FORMAT)))
                .addDeserializer(LocalDate.class, new LocalDateDeserializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT)))
                .addDeserializer(LocalTime.class, new LocalTimeDeserializer(DateTimeFormatter.ofPattern(DEFAULT_TIME_FORMAT)))
                .addSerializer(LocalDateTime.class, new LocalDateTimeSerializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_TIME_FORMAT)))
                .addSerializer(LocalDate.class, new LocalDateSerializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT)))
                .addSerializer(LocalTime.class, new LocalTimeSerializer(DateTimeFormatter.ofPattern(DEFAULT_TIME_FORMAT)));

        //注册功能模块 例如，可以添加自定义序列化器和反序列化器
        this.registerModule(simpleModule);
    }
}
```
```java
    /**
     * 扩展spring mvc框架的消息转换器
     * @param converters
     */
    protected void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        log.info("开始扩展消息转换器...");
        //创建消息转换器列表
        //创建一个消息转换器对象
        MappingJackson2HttpMessageConverter converter = new MappingJackson2HttpMessageConverter();
        //需要为消息转换器设置一个对象转换器，对象转换器可以将java对象序列化为json数据
        converter.setObjectMapper(new JacksonObjectMapper());
        //将消息转换器对象添加到消息转换器列表中
        converters.add(0, converter);
    }
```

3.实现公共字段自动填充（技术点：枚举、注解、AOP、反射）
> 3.1.自定义注解Autofill，用于标识需要进行公共字段自动填充的方法。
> 3.2.自定义切面类AutoFillAspect，用于统一拦截标有@Autofill注解的方法，通过反射为公共字段赋值。
> 3.3.Mapper的方法加入AutoFill注解。

4.店铺营业状态模块
> 4.1. 出现问题：
```java

    /**
     * 获取店铺的营业状态
     *
     * @return
     */
    @ApiOperation("获取店铺的营业状态")
    @GetMapping("/status")
    public Result<Integer> getStatus() {
        Integer status = (Integer) redisTemplate.opsForValue().get(KEY);
        log.info("获取店铺的营业状态:{}", status == 1 ? "营业中" : "打烊中");

        return Result.success(Integer.parseInt(status.toString()));
    }
报错：Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is java.lang.ClassCastException: class java.lang.String cannot be cast to class java.lang.Integer (java.lang.String and java.lang.Integer are in module java.base of loader 'bootstrap')] with root cause

java.lang.ClassCastException: class java.lang.String cannot be cast to class java.lang.Integer (java.lang.String and java.lang.Integer are in module java.base of loader 'bootstrap')



在这段代码处 @PutMapping("/{status}")
    @ApiOperation(value = "设置店铺的营业状态")
    public Result setStatus(@PathVariable Integer status) {
        log.info("设置店铺的营业状态：{}", status == 1 ? "营业中" : "打烊中");

        redisTemplate.opsForValue().set(KEY, status);
        return Result.success();
    }
    
    
报错：Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is java.lang.ClassCastException: class java.lang.Integer cannot be cast to class java.lang.String (java.lang.Integer and java.lang.String are in module java.base of loader 'bootstrap')] with root cause

java.lang.ClassCastException: class java.lang.Integer cannot be cast to class java.lang.String (java.lang.Integer and java.lang.String are in module java.base of loader 'bootstrap')
```

原因：redis配置类里设置了value等序列化器，所以存储的数据都变成了String类型。不设置就可以了，只是在redis中查看数据时，看到的数据是乱码。并不影响使用。