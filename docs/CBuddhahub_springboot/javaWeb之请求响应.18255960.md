# <span style='color:blue'>(原创) 用springboot实现javaWeb对数据的请求响应操作:smile:</span>
***
### springboot开发优点
- 简单
- 方便
- 安全
***
### 一、web知识之请求响应
- 请求：``浏览器发送请求给服务器``
- 响应：``服务器返回响应给浏览器``
- ``请求和响应都是基于HTTP协议的``
***
### 二、web知识之请求参数

#### <span style='color:blue'>1.获取请求参数之简单参数</span>
- <span style='color:orange'>原始方式</span>：
```java
@RequestMapping("/simpleParam")
public String simpleParam(HttpServletRequest request){
    String name = request.getParameter("name");
    String ageStr = request.getParameter("age");
    int age = Integer.parseInt(ageStr);
    //输出
    System.out.println(name);
    System.out.println(age);
    return "success";
}
```
- <span style='color:orange'>简化方式</span>
```java
@RequestMapping("/simpleParam")
public String simpleParam(@RequestParam("name") String name, @RequestParam("age") int age){
    //输出
    System.out.println(name);
    System.out.println(age);
    return "success";
}
```
- <span style='color:orange'>简化方式 <span style='color:red'>springboot方案</span></span>
```java
@RequestMapping("/simpleParam")
public String simpleParam(String name, int age){
    //输出
    System.out.println(name);
    System.out.println(age);
    return "success";
}
```
#### <span style='color:blue'>2.获取请求参数之简单实体类参数</span>
- <span style='color:orange'>原始方式</span>：
```java
@RequestMapping("/entityParam")
public String entityParam(HttpServletRequest request){
    String name = request.getParameter("name");
    String ageStr = request.getParameter("age");
    int age = Integer.parseInt(ageStr);
    //输出
    System.out.println(name);
    System.out.println(age);
    return "success";
}
```
- <span style='color:orange'>简化方式</span>
```java
@RequestMapping("/entityParam")
public String entityParam(User user){
    //输出
    System.out.println(user.getName());
    System.out.println(user.getAge());
    return "success";
}
```
> 总结: 1.原始方式获取请求参数，controller方法中形参声明HttpServletRequest对象，调用对象的getParameter方法获取参数。2.springboot中接受简单参数，请求参数名和方法形参变量名相同，会自动进行类型转换。3.@RequestParam注解可以简化参数名和形参变量名不一致的情况。该注解的required属性默认为true，表示参数必须传递，若为false，表示参数可以不传递。

#### <span style='color:blue'>3.获取请求参数之复杂实体类参数</span>
```java
    @RequestMapping("/complexRequest")
    public String complexRequest(User user) {
        System.out.println(user);
        return "ok";
    }
```
> 注意：1.请求参数与形参对象属性名相同，可直接通过pojo接收，按照对象层次结构关系可接收嵌套的pojo属性参数。

#### <span style='color:blue'>4.获取请求参数之数组集合参数</span>
```java
//数组
    @RequestMapping("/arrayRequest")
    public String arrayRequest(String[] hobby) {
        System.out.println(Arrays.toString(hobby));
        return "ok";
    }

//集合
    @RequestMapping("/listRequest")
    public String listRequest(@RequestParam List<String> hobby) {
        System.out.println(hobby);
        return "ok";
    }
```
> 注意： 1.请求参数与形参对象属性名相同且请求参数为多个，定义数组类型形参即可接收参数。默认情况下，多个请求参数放到数组里。所以想要将多个请求参数放到集合里，需要使用@RequestParam注解的value属性指定请求参数名。

#### <span style='color:blue'>5.获取请求参数之日期时间参数</span>
```java
    @RequestMapping("/dateRequest")
    public String dateRequest(@DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss") LocalDateTime updateTime) {
        System.out.println(updateTime);
        return "ok";
    }
```
> 使用@DateTimeFormat注解，指定日期时间参数的格式。

#### <span style='color:blue'>6.获取请求参数之Json参数</span>
```java
 @RequestMapping("/jsonRequest")
    public String jsonRequest(@RequestBody User user) {
        System.out.println(user);
        return "ok";
    }
```
> 注意：1.请求参数为json格式，使用@RequestBody注解接收参数。

#### <span style='color:blue'>7.获取请求参数之路径参数</span>
```java
    //一个参数
     @RequestMapping("/path/{id}")
    public String pathRequest(@PathVariable Integer id) {
        System.out.println(id);
        return "ok";
    }
    //多个参数
    @RequestMapping("/path/{id}/{name}")
    public String pathRequest(@PathVariable Integer id, @PathVariable String name) {
        System.out.println(id+":"+name);
        return "ok";
    }
```
> 注意：1.请求参数为路径参数，使用@PathVariable注解接收参数。

### 三、web知识之响应参数
#### <span style='color:blue'>1.springboot使用@responseBody响应数据</span>
```java
@RestController
@RequestMapping("/responseBody")
    public String responseBody() {
        return "hello";
    }
```
> 注意：1.springboot中使用@responseBody注解，将返回值作为响应体返回给浏览器。2.要设置统一返回结果（code,msg,data）。
***
> <span style='color:red'>springboot项目的静态资源（前端资源）默认存放目录：</span>
> <span style='color:orange'>1.resources目录下的static目录</span>
> <span style='color:orange'>2.resources目录下的public目录</span>
> <span style='color:orange'>3.resources目录下的resources目录</span>
***






