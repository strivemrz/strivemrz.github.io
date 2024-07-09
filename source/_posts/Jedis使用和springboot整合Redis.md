---
title: Jedis使用和springboot整合Redis
categories: Redis
tags:
  - Redis
excerpt: ''
cover: https://tse3-mm.cn.bing.net/th/id/OIP-C.WvhXlHH4SK25FKGEDkafAgHaEo?pid=ImgDet&rs=1
---

### Jedis操作Redis

**Jedis连接不上服务器**


配置Redis.conf文件

- 注释bind 127.0.0.1
- 给redis添加密码：找到requirepass 123456保存退出
- 连接redis：redis-cli -p 6379 -h localhost -a 123456
- ping测试



Jedis(类似jdbc)是Redis官方推荐的java连接开发工具！使用java操作Jedis中间件,进而操作redis数据库

1. 导入对应依赖

~~~xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>4.2.3</version>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>2.0.2</version>
</dependency>
~~~

2. 编码测试

   ~~~java
   Jedis jedis=new Jedis("服务器ip",6379);
   jedis.auth("123456");//密码
   System.out.println(jedis.ping());//检查是否成功连接
   System.out.println("-------------------------");
   jedis.flushDB();//清空数据库
   System.out.println(jedis.exists("username"));//是否存在key
   jedis.set("username","hello world");
   Set<String> keys = jedis.keys("*");
   System.out.println(keys);
   System.out.println(jedis.del("username"));
   System.out.println(jedis.exists("username"));
   jedis.set("username","hello world");
   System.out.println(jedis.type("username"));
   System.out.println(jedis.randomKey());
   System.out.println(jedis.rename("username", "name"));
   System.out.println(jedis.get("name"));
   jedis.select(0);
   jedis.flushDB();
   System.out.println(jedis.dbSize());
   System.out.println(jedis.flushAll());
   ~~~

   jedis事务操作

   ~~~java
   Jedis jedis=new Jedis("192.168.177.51",6379);
   jedis.auth("123456");
   jedis.flushDB();
   Transaction multi = jedis.multi();
   JSONObject jsonObject=new JSONObject();//fastjson
   jsonObject.put("name","xiaohong");
   jsonObject.put("pass","xiaolv");
   try {
       String s = jsonObject.toString();
       System.out.println(s);
       multi.set("user",s);
       int i=1/0;
       multi.exec();
   }catch (Exception e){
       multi.discard();//取消事务
       System.out.println(jedis.get("user"));
       e.printStackTrace();
   }finally {
       multi.close();
   }
   ~~~

   ### springboot集成Redis

   1. 说明：在springboot2.x之后，原来使用的jedis被替换为了lettuce

      jedis：采用的直连，多个线程操作的话是不安全的，如果想要避免不安全的，使用jedis pool连接池！更像BIO模式

      lettuce：采用netty，实例可以再多个线程中进行共享，不存在线程不安全的情况！可以减少线程数据，更像NIO模式

      ```java
      @Bean
      @ConditionalOnMissingBean(
          name = {"redisTemplate"}
      )
      @ConditionalOnSingleCandidate(RedisConnectionFactory.class)
      public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
          RedisTemplate<Object, Object> template = new RedisTemplate();
          template.setConnectionFactory(redisConnectionFactory);
          return template;
      }
      
      @Bean
      @ConditionalOnMissingBean
      @ConditionalOnSingleCandidate(RedisConnectionFactory.class)
      public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) {//Redis中String是常用类型，所以单独提出来一个bean
          return new StringRedisTemplate(redisConnectionFactory);
      }
      ```

   2. 整合步骤

      - 导入依赖

        ~~~xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        ~~~

      - 配置连接

        ```properties
        spring.redis.host=192.168.177.51##服务器主机号
        spring.redis.port=6379##端口号
        spring.redis.password=123456##密码
        ```

      - 代码测试
      ```java
        @Autowired
        private RedisTemplate redisTemplate;
        
        @Test
        void contextLoads() {
            redisTemplate.opsForValue().set("user","小红");
            redisTemplate.opsForValue().get("user");
        }
      ```

   

### 自定义RedisTemplate

1. 存储对象

```java
@Test
public void test1() throws JsonProcessingException {
    Admin admin=new Admin("小芳","111");
    String s = new ObjectMapper().writeValueAsString(admin);
    System.out.println(s);
    redisTemplate.opsForValue().set("user",s);
    System.out.println(redisTemplate.opsForValue().get("user"));
}
```

2. 自定义RedisTemplate

~~~java

@Configuration
public class RedisConfig {
    @Bean
    @SuppressWarnings("all")
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {

        RedisTemplate<String, Object> template = new RedisTemplate<String, Object>();
        template.setConnectionFactory(factory);

        // 序列化配置 解析任意对象
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        // json序列化利用ObjectMapper进行转义
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        // 2.序列化String类型
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();

        // key采用String的序列化方式
        template.setKeySerializer(stringRedisSerializer);
        // hash的key也采用String的序列化方式
        template.setHashKeySerializer(stringRedisSerializer);
        // value序列化方式采用jackson
        template.setValueSerializer(jackson2JsonRedisSerializer);
        // hash的value序列化方式采用jackson
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();

        return template;
    }
}
~~~