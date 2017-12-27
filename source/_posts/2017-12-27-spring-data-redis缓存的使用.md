---
title: spring-data-redis缓存的使用
date: 2017-12-27 16:53:23
tags: redis
categories: spring
---

公司最近需要将一个老项目升级，其中涉及到缓存，从memcached到Redis。
* redis与 memcached相比，redis支持key-value数据类型，同事支持list、set、hash等数据结构的存储。
* redis支持数据的备份，即master-slave模式的数据备份。
* redis支持数据的持久化。
* redis在很多方面支持数据库的特性，可以这样说他就是一个数据库系统，而memcached只是简单地K/V缓存。
* 它们在性能方面差别不是很大，读取方面尤其是针对批量读取性能方面memcached占据优势。当然redis也有他的优点，如持久性、支持更多的数据结构。
* 所以在选择方面如果有持久方面的需求或对数据类型和处理有要求的应该选择redis。

如果简单的key/value 存储应该选择memcached

<!-- more -->

# jedis
首先通过maven引入jedis的依赖：
```
<!-- 注意：2.9.0以后版本才支持集群，且该版本集群密码有问题 -->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
```
创建Jedis对象，调用set方法，并通过get方法获取到值并打印：
```java
Jedis jedis = new Jedis("localhost", 6379);
jedis.set("singleJedis", "hello jedis!");
System.out.println(jedis.get("singleJedis"));
jedis.close();
```
需要自己实现一些jedis的工具类，使用不方便。

# spring-data-redis

spring已经将redis集成，并提供了较为友好的redisTemplate访问redis。这里配置集群环境
## spring集成
pom加入：
```
<!-- jedis （一个redis client端的jar）-->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
<!-- spring-data-redis 依赖-->
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>1.7.1.RELEASE</version>
</dependency>
```
添加配置文件redis.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://www.springframework.org/schema/beans" xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:task="http://www.springframework.org/schema/task" xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:cache="http://www.springframework.org/schema/cache"
    xsi:schemaLocation="
    http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.1.xsd
    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.1.xsd
    http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.1.xsd
    http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.1.xsd
    http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task-4.1.xsd
    http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.1.xsd
    http://www.springframework.org/schema/cache http://www.springframework.org/schema/cache/spring-cache.xsd">

     <!--  redis连接池  这里引用的是jedis 包中的功能  -->
    <bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
        <property name="maxTotal" value="${redis.maxActive:1024}" />
        <property name="maxIdle" value="${redis.maxIdle:1024}" />
        <property name="maxWaitMillis" value="${redis.maxWait:10000}" />
        <property name="testOnBorrow" value="${redis.testOnBorrow:true}" />
        <property name="testOnReturn" value="${redis.testOnReturn:true}" />
    </bean>

    <!-- Redis集群配置     这里使用的是spring-data-redis  包中内容 -->
     <bean id="redisClusterConfig" class="org.springframework.data.redis.connection.RedisClusterConfiguration">
        <property name="maxRedirects" value="6"></property>
        <property name="clusterNodes">
            <set>
                <bean class="org.springframework.data.redis.connection.RedisNode">
                    <constructor-arg name="host" value="192.168.1.105"></constructor-arg>
                    <constructor-arg name="port" value="7111"></constructor-arg>
                </bean>

                <bean class="org.springframework.data.redis.connection.RedisNode">
                    <constructor-arg name="host" value="192.168.1.105"></constructor-arg>
                    <constructor-arg name="port" value="7112"></constructor-arg>
                </bean>
                <bean class="org.springframework.data.redis.connection.RedisNode">
                    <constructor-arg name="host" value="192.168.1.105"></constructor-arg>
                    <constructor-arg name="port" value="7116"></constructor-arg>
                </bean>
                <bean class="org.springframework.data.redis.connection.RedisNode">
                    <constructor-arg name="host" value="192.168.1.102"></constructor-arg>
                    <constructor-arg name="port" value="7113"></constructor-arg>
                </bean>
                 <bean class="org.springframework.data.redis.connection.RedisNode">
                    <constructor-arg name="host" value="192.168.1.102"></constructor-arg>
                    <constructor-arg name="port" value="7114"></constructor-arg>
                </bean>
                 <bean class="org.springframework.data.redis.connection.RedisNode">
                    <constructor-arg name="host" value="192.168.1.102"></constructor-arg>
                    <constructor-arg name="port" value="7115"></constructor-arg>
                </bean>
            </set>
        </property>
    </bean> 
    <!-- Redis连接工厂     -->
    <bean id="redis4CacheConnectionFactory"
        class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
        <constructor-arg name="clusterConfig" ref="redisClusterConfig" />
        <property name="timeout" value="${redis.timeout:10000}" />
        <property name="poolConfig" ref="jedisPoolConfig" />
    </bean>
    <!-- 存储序列化 -->
    <bean name="stringRedisSerializer"
        class="org.springframework.data.redis.serializer.StringRedisSerializer" />

    <!-- 集群Resis使用模板 -->
    <bean id="clusterRedisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
        <property name="connectionFactory" ref="redis4CacheConnectionFactory" />
        <property name="keySerializer" ref="stringRedisSerializer" />
        <property name="hashKeySerializer" ref="stringRedisSerializer" />
        <property name="valueSerializer" ref="stringRedisSerializer" />
        <property name="hashValueSerializer" ref="stringRedisSerializer" />
    </bean>

</beans>
```

## springboot集成
pom加入：
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
添加ReisConfig.java配置文件
```java
/**
 * @author ltlixer
 * @date 2017/12/7 上午11:02
 */
@Slf4j
@Configuration
@EnableCaching
public class RedisConfig extends CachingConfigurerSupport {

    @Bean
    @Override
    public KeyGenerator keyGenerator() {
        return new KeyGenerator() {
            @Override
            public Object generate(Object target, Method method, Object... params) {
                StringBuilder sb = new StringBuilder();
                sb.append(target.getClass().getName());
                sb.append(method.getName());
                for (Object obj : params) {
                    sb.append(obj.toString());
                }
                return sb.toString();
            }
        };
    }

    @SuppressWarnings("rawtypes")
    @Bean
    public CacheManager cacheManager(RedisTemplate redisTemplate) {
        RedisCacheManager rcm = new RedisCacheManager(redisTemplate);
        //设置缓存过期时间,1个月
        rcm.setDefaultExpiration(60 * 60 * 24 * 30);
        return rcm;
    }

    @Bean
    public RedisTemplate<String, String> redisTemplate(RedisConnectionFactory factory) {
        StringRedisTemplate template = new StringRedisTemplate(factory);
        //自定义redis序列化方式
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        template.setValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();
        return template;
    }

    /**
     * redis异常处理器
     * 默认SimpleCacheErrorHandler是抛出异常，此处自定义处理方式不抛出
     *
     * @return
     */
    @Bean
    @Override
    public CacheErrorHandler errorHandler() {
        return new CacheErrorHandler() {
            @Override
            public void handleCacheGetError(RuntimeException e, Cache cache, Object key) {
                log.error("cache get error", e);
            }

            @Override
            public void handleCachePutError(RuntimeException e, Cache cache, Object key, Object value) {
                log.error("cache put error", e);
            }

            @Override
            public void handleCacheEvictError(RuntimeException e, Cache cache, Object key) {
                log.error("cache evict error", e);
            }

            @Override
            public void handleCacheClearError(RuntimeException e, Cache cache) {
                log.error("cache clear error", e);
            }
        };
    }
}

```
注意：需要使用相同的序列化方式 
spring-data-redis提供了若干个Serializer，主要包括：
* JacksonJsonRedisSerializer
* JdkSerializationRedisSerializer
* OxmSerializer