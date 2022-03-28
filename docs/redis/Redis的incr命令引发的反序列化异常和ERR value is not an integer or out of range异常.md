## `Redis`的`incr`命令引发的反序列化异常和`ERR value is not an integer or out of range`异常

最近在开发中，使用Redis来实现自增ID。为什么使用Redis？这是一个高并发访问，需要考虑操作冲突导致数据不一致的问题。而Redis是内存型存储，相比关系型数据库，操作更快，避免了频繁的文件写操作。更重要的是，Redis中有个INCR和INCRBY命令，都可以实现值递增的原子性操作，方便了解决了高并发时的冲突问题。

### redis命令说明

`incr`：

- 对存储在指定`key`的数值执行原子的加1操作。

- 如果指定的key不存在，那么在执行incr操作之前，会先将它的值设定为`0`。

- 如果指定的key中存储的值不是字符串类型（fix：）或者存储的字符串类型不能表示为一个整数，那么执行这个命令时服务器会返回一个错误(eq:(error) ERR value is not an integer or out of range)。

### 序列化异常

#### 摸索

当我执行如下命令时：

```java
public void contextLoads1() {
        RedisTemplate redisTemplate = RedisUtil.getRedis();
        ValueOperations valueOperations = redisTemplate.opsForValue();
  //键为 key1
  Long key1 = valueOperations.increment("key1");
        log.info("执行increment(\"key1\")返回值是：{}",key1);
    }
```

执行结果如下：成功返回了递增后的结果

![image-20220328151648657](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202203281516700.png)

我当时要在别处查看这个`key1`键对应的值是多少，然后就用以下命令查看，问题来了：

![image-20220328151947826](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202203281519865.png)

居然返回的是null，我就纳闷了，明明前面执行的`increment`命令，为啥返回为null，我就有如下**几个猜想**：

- 猜想一：是不是`increment`命令执行失败了，服务器上不存在`key1

于是乎，我就打开服务器查看，发现存在啊！！！

![image-20220328152228745](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202203281522780.png)

- 猜想二：代码中`get`命令没有获取到值

经过各种debug，最后发现是前人埋下的一个坑，问题出现在如下代码里面：

```java
@Bean
public RedisTemplate redisTemplate(RedisTemplate template) {
    template.setKeySerializer(new StringRedisSerializer());
   //ValueSerializer使用的是JdkSerializationRedisSerializer，并且默认情况下就是JdkSerializationRedisSerializer，我也不知道前人是什么个意思，再重新设置同样的序列化
    template.setValueSerializer(new JdkSerializationRedisSerializer() {
        @Override
        public Object deserialize(byte[] bytes) {
            try {
              //最夸张的是直接调用super.deserialize方法 ！！！
                return super.deserialize(bytes);
            } catch (Exception e) {
              //最最最夸张的是直接返回null ！！！
                return null;
            }
        }
    });
    return template;
}
```

**问题原因**：在反序列化异常的时候，直接返回null，把deserialize方法抛出的异常捕获，直接返回，也不打印错误日志，这不是误导我嘛！！！有错误就抛啊，我还在那里以为`increment`自增失败了呢。。。

#### 解决

可是为什么出现了序列化异常呢？？？带着大大的疑问继续往下走：

我们先去服务器上通过命令查看`key1`对应值的编码类型：

![image-20220328151344766](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202203281513808.png)

可以看到值是一个`int`类型，这时候我就想到了前面"大神"配置的`ValueSerializer`值是`JdkSerializationRedisSerializer`，我们先改下代码，让异常打印出来：

```java
@Bean
public RedisTemplate redisTemplate(RedisTemplate template) {
    template.setKeySerializer(new StringRedisSerializer());
    template.setValueSerializer(new JdkSerializationRedisSerializer() {
        @Override
        public Object deserialize(byte[] bytes) {
            try {
                return super.deserialize(bytes);
            } catch (Exception e) {
                log.error("deserialize 反序列化异常：{}",e.getMessage());
                return "deserialize 反序列化异常：{" + e.getMessage() + "}";
                //return null;
            }
        }
    });
    return template;
}
```

再次执行获取的方法

```java
@Test
public void contextLoads3() {
    RedisTemplate redisTemplate = RedisUtil.getRedis();
    ValueOperations valueOperations = redisTemplate.opsForValue();
    Object value1 = valueOperations.get("key1");
    log.info("键为key1的返回值是：{}",value1);
}
```

出现如下错误：

```java
2022-03-28 15:42:28.255 -ERROR 6217 [] {magenta} --- [           main] ybplan.config.RedisConfig                : deserialize 反序列化异常：Cannot deserialize; nested exception is org.springframework.core.serializer.support.SerializationFailedException: Failed to deserialize payload. Is the byte array a result of corresponding serialization for DefaultDeserializer?; nested exception is java.io.EOFException
--2022-03-28 15:42:28.255 - INFO 6217 [] {magenta} --- [           main] ybplan.SxapiApplicationTest              : 键为key1的返回值是：deserialize 反序列化异常：{Cannot deserialize; nested exception is org.springframework.core.serializer.support.SerializationFailedException: Failed to deserialize payload. Is the byte array a result of corresponding serialization for DefaultDeserializer?; nested exception is java.io.EOFException}
-
```

关键在于如下`Is the byte array a result of corresponding serialization for DefaultDeserializer?`,

`byte array`？？？,然后我就去官网查了下资料发现在`JdkSerializationRedisSerializer`有如下两个方法`deserialize` `serialize`

官网解释如下：

![image-20220328155146044](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202203281551074.png)

![image-20220328155129556](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202203281551586.png)

存储的时候将数据存储为`binary data`，获取的时候通过反序列化将`binary data`转为`Object`，可是我前面是直接通过`incrment`设置的值的，我们在看下`incrment的源码，

```java
/*
 * (non-Javadoc)
 * @see org.springframework.data.redis.core.ValueOperations#increment(java.lang.Object)
 */
@Override
public Long increment(K key) {

   byte[] rawKey = rawKey(key);
   return execute(connection -> connection.incr(rawKey), true);
}
```

从源码中发现，当我们直接调用increment方法，递增量的初始值是由Redis生成的，根本没有走`JdkSerializationRedisSerializer`的序列化策略，又何来序列化，所以说问题原因找到了。为了证明这一点我们做如下测试：

```java
@Test
public void contextLoads3() {
    RedisTemplate redisTemplate = RedisUtil.getRedis();
    ValueOperations valueOperations = redisTemplate.opsForValue();
    //用JdkSerializationRedisSerializer序列化去设置值和获取值
    valueOperations.set("key2",1);
    Object value2 = valueOperations.get("key2");
    log.info("键为key2的返回值是：{}",value2);
}
```

得到的结果如下：

![image-20220328160125548](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202203281601600.png)

#### 结论

所以问题原因找到了，问题就是在序列化为`JdkSerializationRedisSerializer`的情况下先使用`incrment`，再去使用get去获取会出现

`Cannot deserialize`异常，因为`incrment`设置的值本来就不是一个`binary data`，所以在获取的时候失败了。



### ERR value is not an integer or out of range

#### 摸索

在键为`key2`的值上执行`increment`

```java
@Test
public void contextLoads3() {
    RedisTemplate redisTemplate = RedisUtil.getRedis();
    ValueOperations valueOperations = redisTemplate.opsForValue();
    valueOperations.set("key2",1);
    log.info("在使用set设置键为key2的键上执行increment，获取到的值是：{}",valueOperations.increment("key2"));
   
}
```

出现如下异常：

```java
org.springframework.data.redis.RedisSystemException: Error in execution; nested exception is io.lettuce.core.RedisCommandExecutionException: ERR value is not an integer or out of range
```

根据前面`Redis`小节下的`incr`命令说可知，`key2`对应的值不是字符串类型或者存储的字符串类型不能表示为一个整数

我们通过服务器来验证一下：

我们先通过`get`获取,发现报错

![image-20220328161205943](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202203281612991.png)

查看下编码类型

![image-20220328161300489](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202203281613537.png)

通过前面可知`key1`的编码类型是`int`，同样的值是1，因为序列化的原因，导致的编码类型不同,并且服务器上并没有反序列化，就直接获取`key2`的值，所以报错了。

由`object encoding key2`返回的`raw`可知（具体的raw的含义会在下文给出解释），当我们对`key2`执行`incrment`的时候，因为值并不是一个字符串类型或者存储的字符串类型不能表示为一个整数，所以自增会出现`ERR value is not an integer or out of range`错误。

### 拓展

[redis的字符串对象和字符串的编码类型](https://blog.csdn.net/woshiyangyunlong/article/details/108730447)

### 推荐书籍

《Redis设计与实现》黄建宏

[思维导图](https://www.processon.com/view/link/624172a8f346fb072d11a3f5)