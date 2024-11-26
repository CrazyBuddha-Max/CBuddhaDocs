---
comments: true
---
（原创）Redis教程
***
## 一、定义与特性
> 基于内存的key-value结构数据库。（key是字符串类型，value有五种常用的数据类型）

> redis是单线程的，但是因为是基于内存的，所以读写速度非常快。
> redis支持持久化，可以将内存中的数据持久化到硬盘上。
> redis支持主从复制，可以实现数据的备份和负载均衡。
> redis支持事务，可以保证数据的一致性。
> redis支持发布/订阅，可以实现消息的实时推送。
> redis支持Lua脚本，可以实现复杂的业务逻辑。
> redis支持多种编程语言的客户端，如java、python、php等。

> 不区分大小写。

## 二、五种常用的数据类型
> 1. String（字符串）
> 2. Hash（哈希,类似java的hashmap）
> 3. List（列表，可以有重复元素，类似java的linkedlist）
> 4. Set（集合，没有重复元素，类似java的hashset）
> 5. ZSet/soorted set（有序集合，没有重复元素，每个元素对应一个分数，根据分数升序排序）

## 三、常用命令
### 1. 字符串操作命令
```
SET key VALUE 设置指定key的value值
GET key 获取指定key对应的value值
SETEX key seconds value 设置指定key的value值，并将key的过期时间设为seconds秒
SETNX key value 只有在key不存在时，才设置key的value值
```

### 2. 哈希操作命令
```
HSET key field value 设置指定key的field字段的value值
HGET key field 获取指定key的field字段的value值 
HDEL key field 删除指定key的field字段
HKEYS key 获取指定key的所有field字段
HVALS key 获取指定key的所有value值
```

### 3. 列表操作命令
```
LPUSH key value1 [value2] 将一个或多个值插入到列表头部
LRANGE key start stop 获取列表指定范围内的元素
RPOP key 移除并返回列表最后一个元素
LLEN key 获取列表长度
```

### 4. 集合操作命令
```
SADD key member1 [member2] 向集合中添加一个或多个成员
SMEMBERS key 获取集合中所有成员
SREM key member1 [member2] 移除集合中一个或多个成员
SCARD key 获取集合中成员的个数
SINTER key1 key2 获取两个集合的交集
SUNION key1 key2 获取两个集合的并集
```

### 5. 有序集合操作命令
```
ZADD key score member 添加一个或多个成员，或者更新已存在成员的分数
ZRANGE key start stop [WITHSCORES] 获取有序集合中指定范围内的成员 //默认升序排列
ZREM key member 移除有序集合中的一个或多个成员
ZCARD key 获取有序集合中成员的个数
ZSCORE key member 获取有序集合中指定成员的分数
ZINCRBY key increment member 有序集合中指定成员的分数增加increment
```

### 6. 通用命令
```
KEYS pattern 查找所有符合给定模式pattern的key
EXISTS key 检查给定key是否存在 //1表示存在，0表示不存在
DEL key 删除指定key
TTL key 获取指定key的过期时间（秒）
PERSIST key 移除指定key的过期时间，使其成为永久key
TYPE key 获取指定key的数据类型
```

## 四、在java中操作Redis
### 1.Redis的Java客户端
> Jedis(生态完整)、Lettuce(性能高效)、Redisson,Spring Data Redis(spring提供，封装了Jedis和Lettuce)等。

### 2.Spring Data Redis的使用
#### 1. 引入依赖
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
#### 2. 配置Redis数据源
```
spring.redis.host=127.0.0.1
spring.redis.port=6379
```
#### 3. 编写配置类，创建RedisTemplate对象
```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@Configuration
@Slf4j
public class RedisConfiguration {
    @Bean
    public RedisTemplate redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        log.info("开始创建redis模板对象...");
        RedisTemplate redisTemplate = new RedisTemplate();
        //设置连接工厂对象
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        //设置key的序列化器、
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        //返回对象
        return redisTemplate;
    }
}
```
#### 4. 通过RedisTemplate对象操作Redis
```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.core.*;

@SpringBootTest
public class SpringDataRedisTest {

    @Autowired
    private RedisTemplate redisTemplate;

    @Test
    public void testRedisTemplate() {
        System.out.println(redisTemplate);
        ValueOperations valueOperations = redisTemplate.opsForValue();
        HashOperations hashOperations = redisTemplate.opsForHash();
        ListOperations listOperations = redisTemplate.opsForList();
        SetOperations setOperations = redisTemplate.opsForSet();
        ZSetOperations zSetOperations = redisTemplate.opsForZSet();

    }
}
```