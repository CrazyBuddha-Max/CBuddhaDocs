（原创）mybatis教程
***
## 1. 定义

     mybatis是一个持久层框架(数据访问层)，用于简化JDBC的开发。
***
## 2.配置SQL语句

- 方案一：使用注解。
- 方案二：配置XML映射文件。

***
## 3.JDBC与mybatis对比：
### JDBC
  - > 1.需要自己写SQL语句，需要手动处理连接和事务管理。
  - > 2.频繁的获取连接释放连接导致资源浪费与性能降低。mybatis提供了数据库连接池技术来解决该问题。
  - > 3.数据库连接需要手动处理，繁琐。
  - > 4.SQL语句写在代码中，造成代码不易维护，实际应用SQL变化的可能较大，sql变动需要改变java代码。
### mybatis
  - > 1.只需要编写SQL语句和映射关系，不需要手动处理连接和事务管理。
  - > 2.mybatis使用ORM思想将数据库记录映射到java对象中，可以省去很多手动设置结果集的操作。
  - > 3.使用mybatis可以避免SQL注入问题。
  - > 4.mybatis使用xml文件配置SQL语句，可以方便地管理SQL语句。如下是mybatis的数据库连接的解决方案。

    ![img](https://img2023.cnblogs.com/blog/3467365/202406/3467365-20240629222550451-285971511.png)
***
## 4.数据库连接池：
### 定义
  - > 一个负责分配、管理数据库连接的容器。
### 优点
  - > 1.提高数据库连接的效率。
  - > 2.避免数据库连接资源浪费。
  - > 3.便于数据库连接的管理。
### 接口
  - > 1.DataSource：数据源接口，用于获取数据库连接。（最关键接口，自主开发连接池必须实现该接口）
  - > 2.Connection：数据库连接接口，用于执行SQL语句。
  - > 3.Statement：SQL语句执行接口，用于执行SQL语句。
  - > 4.PreparedStatement：预编译SQL语句执行接口，用于执行SQL语句。
  - > 5.ResultSet：结果集接口，用于获取查询结果。
### 实现
  - > 1.DBCP：Apache提供的数据库连接池。
  - > 2.C3P0：C3P0是hibernate提供的一个数据库连接池。
  - > 3.Druid：Druid是阿里巴巴提供的数据库连接池。
  - > 4.Hikari：Hikari是Spring Boot 2.0之后默认的数据库连接池。
***
## 5.lombok
### 定义
  - > 一个Java库，用于简化实体类开发。
### 优点
  - > 解决了实体类实现的繁琐问题。
### 实现
  - > 1.@Getter：用于生成getter方法。
  - > 2.@Setter：用于生成setter方法。
  - > 3.@ToString：用于生成toString方法。
  - > 4.@EqualsAndHashCode：用于生成equals和hashCode方法。
  - > 5.@NoArgsConstructor：用于生成无参构造方法。
  - > 6.@AllArgsConstructor：用于生成全参构造方法。
  - > 7.@Data：用于生成getter、setter、toString、equals和hashCode方法。
  - > 8.@Builder：用于生成建造者模式。
  - > 9.@Slf4j：用于生成日志对象。
  - > 10.@Log4j2：用于生成日志对象。

## 6.基于注解的方案实现基础SQL语句
### 删除
  - > 1.根据主键id删除数据。@Delete("DELETE FROM user WHERE id = #{id}")
  - > 2.根据条件删除数据。
  - > 配置mybatis日志，需要在application.yml中配置。（预编译SQL：性能高，更安全防止sql注入）
  - > mybatis提供参数占位符：1.生成预编译sql，参数传递 #{}。2. 拼接sql ${}。
### 增加
  - > 1.插入一条数据并返回主键id需要的注解：@Insert("INSERT INTO user(x,y) VALUES(#{x},#{y})") @Options("useGeneratedKeys=true,keyProperty=id")
### 修改
  - > 1.根据主键id修改数据。@Select()
  - > 2.根据条件修改数据。
### 查询
  - > 1.根据id查询数据。
  - > 2.根据条件查询数据。
  - > 实体类属性值和数据库查询返回的字段名一致，mybatis会自动封装。不一致不能封装，也就查询不到该属性。解决方案一：给字段起别名，让其与实体类属性名一致。解决方案二：使用@Results,@Result注解手动进行映射封装（麻烦不用）。方案三：开启mybatis驼峰命名的自动开关。在application.yml中加入mybatis.configuration.map-underscore-to-camel-case=true（注意：数据库字段名是由下划线连接命名，java属性名由驼峰命名）
  - > 在模糊查询中，使用like关键字会遇到字符拼接，需要使用字符串拼接函数concat('%',#{value},'%')，防止SQL注入。（在springboot1.x和单独使用mybatis时，需要在参数名前加上@Param注解并声明方法形参名称，因为早期版本对mapper接口编译时不会保留方法的形参名称，新版本会内置编译插件对接口方法编译时会将方法形参名称保留）
***
## 7.基于XML映射文件实现基础SQL语句
### 规范
  - > 1.同包同名：在resources文件下创建一个和Mapper接口所在相同包名的文件夹(com.example.mapper)，然后在该文件夹下创建XML映射文件，XML映射文件名和Mapper接口名(例如EmpMapper)一致。
  - > 2.XML映射文件中的namespace属性和Mapper接口全类名（全限定名）一致(例如com.itheima.mapper.EmpMapper)。
  - > 3.XML映射文件中sql语句的id属性和Mapper接口中的方法名(例如getEmpByCondition)一致,返回类型要与单条数据封装的全类名(例如com.example.pojo.Epm)一致。
> 使用注解来映射简单语句会使代码显得更加简洁，但对于稍微复杂一点的语句，Java 注解不仅力不从心，还会让本就复杂的 SQL 语句更加混乱不堪。 因此，如果你需要做一些很复杂的操作，最好用 XML 来映射语句。选择何种方式来配置映射，以及是否应该要统一映射语句定义的形式，完全取决于你和你的团队。换句话说，永远不要拘泥于一种方式，你可以很轻松地在基于注解和 XML 的语句映射方式间自由移植和切换。
***
## 8.动态sql
### 定义
  - > 随着用户的输入或者外部条件的变化而变化的语句叫做动态sql。
### 实现
  - > 1.if标签：用于判断条件是否成立，成立则执行if标签内的sql语句。
  - > 2.where标签：1.动态生成where关键字。2.自动去掉第一个and关键字。
  - > 3.set标签：1.将所有更新字段进行包裹。2.自动去掉最后一个逗号。
  - > 4.foreach标签：用于遍历集合，可以实现批量插入和批量删除。foreach collection="遍历的集合" item="遍历出来的元素" separator="分隔符" open="遍历开始前拼接的片段" close="遍历结束后拼接的片段"
  - > 5.sql和include标签：sql标签可以将一些公共的sql语句抽取出来，使用时直接用include标签引用即可。
    ```mysql
    <sql id="commonSelect">
            select id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time
            from emp
        </sql>
      <include refid="commonSelect"></include>
    ```
***
## 9.分页插件PageHelper
### 定义
  - > 分页插件，用于实现分页功能。
### 实现
  - > 1.引入依赖。
  - > 2.在application.yml中配置分页插件。
  - > 3.使用PageHelper.startPage(当前页,每页显示的条数)。
  - > 4.使用PageInfo(page)获取分页信息。
***
## 10.mybatis-plus
### 定义
  - > 基于mybatis的增强工具，简化开发。
### 实现
未完待续...
***