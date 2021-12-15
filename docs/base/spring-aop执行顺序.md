# spring-aop

## 通知（Advice）类型的说明

@Before 前置通知（Before advice） ：

> 在某连接点（JoinPoint）——核心代码（类或者方法）之前执行的通知，但这个通知不能阻止连接点前的执行。为啥不能阻止线程进入核心代码呢？因为@Before注解的方法入参不能传ProceedingJoinPoint，而只能传入JoinPoint。要知道从aop走到核心代码就是通过调用ProceedingJionPoint的proceed()方法。而JoinPoint没有这个方法。
> 这里牵扯区别这两个类：Proceedingjoinpoint 继承了 JoinPoint 。是在JoinPoint的基础上暴露出 proceed 这个方法。proceed很重要，这个是aop代理链执行的方法。暴露出这个方法，就能支持 aop:around 这种切面（而其他的几种切面只需要用到JoinPoint，这跟切面类型有关）， 能决定是否走代理链还是走自己拦截的其他逻辑。建议看一下 JdkDynamicAopProxy的invoke方法，了解一下代理链的执行原理。这样你就能明白 proceed方法的重要性。

@After 后通知（After advice） ：

> 当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）。

@AfterReturning 返回后通知（After return advice） ：

> 在某连接点正常完成后执行的通知，不包括抛出异常的情况。

@Around 环绕通知（Around advice） ：

> 包围一个连接点的通知，类似Web中Servlet规范中的Filter的doFilter方法。可以在方法的调用前后完成自定义的行为，也可以选择不执行。这时aop的最重要的，最常用的注解。用这个注解的方法入参传的是ProceedingJionPoint pjp，可以决定当前线程能否进入核心方法中——通过调用pjp.proceed();

@AfterThrowing 抛出异常后通知（After throwing advice） ：

>  在方法抛出异常退出时执行的通知。

## 执行流程图

![](https://gitee.com/laoyouji1018/images/raw/master/img/202112041750576.png)

## 测试

### @AfterReturning与@AfterReturning

#### 定义controller

```java
/**
controller
     */
@ApiOperation("测试AfterReturning")
    @PostMapping("test")
    public String test(@RequestBody JSONObject json) {
        return myPlanService.test(json);
    }

```

#### 定义service

```java
/**
    service
     */    
    public String test(JSONObject json) {
         return "test";
    }
```

#### 定义切入方法

```java
/**
    切面
     */
@AfterReturning(returning = "rvt", pointcut = "execution(* ybplan.service.MyPlanService.test(..))")
    public Object test(JoinPoint joinPoint, Object rvt) {
        Map<String, Object> argMap = AopUtil.getArgMap(joinPoint);
        JSONObject param = (JSONObject) argMap.get("json");
        log.info("正常test————》》参数：{}", param);
        log.info("正常test————》》返回值：{}", rvt);
        return joinPoint;
    }

@AfterThrowing(throwing = "e",pointcut = "execution(* ybplan.service.MyPlanService.test(..))")
    public Object test2(JoinPoint joinPoint,Exception e) {
        Map<String, Object> argMap = AopUtil.getArgMap(joinPoint);
        JSONObject param = (JSONObject) argMap.get("json");
        log.info("异常test2————》》参数：{}", param);
        log.info("异常test2————》》异常：{}", e.getMessage());
        return joinPoint;
    }
```

#### 工具类

```java
/**
  获取参数的工具类
     */   
public static Map<String, Object> getArgMap(JoinPoint joinPoint) {
        String[] params = (new DefaultParameterNameDiscoverer()).getParameterNames(getMethod(joinPoint));
        Map<String, Object> map = new TreeMap();
        if (params == null) {
            return map;
        } else {
            for(int i = 0; i < params.length; ++i) {
                map.put(params[i], joinPoint.getArgs()[i]);
            }

            return map;
        }
    }
```

#### 测试结果

当我们切入的方法是没有抛出异常时：

> 执行test方法！！！
>
> 正常Test————》》参数：{"additionalProp1":{},"additionalProp3":{},"additionalProp2":{}}
>
> 正常Test————》》返回值：test

当我们切入的方法抛出异常时：

```java
 public String test(JSONObject json) {
        int a = 1 / 0;
        return "test";
    }
```

> 执行test方法！！！
>
> 异常test2————》》参数：{"additionalProp1":{},"additionalProp3":{},"additionalProp2":{}}
>
> 异常test2————》》异常：/ by zero

### @Before

#### 定义切入方法

```java
@Before(value = "execution(* ybplan.service.MyPlanService.test(..))")
    public Object test4(JoinPoint joinPoint) {
        Map<String, Object> argMap = AopUtil.getArgMap(joinPoint);
        JSONObject param = (JSONObject) argMap.get("json");
        log.info("test4————》》参数：{}", param);
        log.info("test4————》》joinPoint：{}", joinPoint);
        return joinPoint;
    }
```

#### 测试结果

当我们切入的方法是没有抛出异常时：

> test4————》》参数：{"additionalProp1":{},"additionalProp3":{},"additionalProp2":{}}
>
> test4————》》joinPoint：execution(String ybplan.service.MyPlanService.test(JSONObject))
>
> 执行test方法！！！

当我们切入的方法抛出异常时：

```java
 public String test(JSONObject json) {
        int a = 1 / 0;
        return "test";
    }
```

> test4————》》参数：{"additionalProp1":{},"additionalProp3":{},"additionalProp2":{}}
>
> test4————》》joinPoint：test
>
> 执行test方法！！！

### @Around

#### 定义切入方法

```java
@Around(value = "execution(* ybplan.service.MyPlanService.test(..))")
    public Object test3(ProceedingJoinPoint joinPoint) throws Throwable {
        Map<String, Object> argMap = AopUtil.getArgMap(joinPoint);
        JSONObject param = (JSONObject) argMap.get("json");
        log.info("test3————》》参数：{}", param);
        log.info("随意打印一些数据");
        log.info("test3————》》返回值：{}", joinPoint.proceed());
        return joinPoint.proceed();
    }
```

#### 测试结果

当我们切入的方法是没有抛出异常时：

> test3————》》参数：{"additionalProp1":{},"additionalProp3":{},"additionalProp2":{}}
>
> 执行test方法！！！
>
> test3————》》返回值：test
>
> 执行test方法！！！

当我们切入的方法抛出异常时：

```java
 public String test(JSONObject json) {
        int a = 1 / 0;
        return "test";
    }
```

> test3————》》参数：{"additionalProp1":{},"additionalProp3":{},"additionalProp2":{}}
>
> 随意打印一些数据。
>
> 执行test方法！！！

由上方的结果发现：执行了`log.info("随意打印一些数据");`可以看出在执行`joinPoint.proceed()`方法之前，前面的方法都会执行

### @After

#### 定义切入方法

```java
@After(value = "execution(* ybplan.service.MyPlanService.test(..))")
    public Object test5(JoinPoint joinPoint) {
        Map<String, Object> argMap = AopUtil.getArgMap(joinPoint);
        JSONObject param = (JSONObject) argMap.get("json");
        log.info("test4————》》参数：{}", param);
        log.info("test4————》》joinPoint：{}", joinPoint);
        return joinPoint;
    }
```

#### 测试结果

当我们切入的方法是没有抛出异常时：

> 执行test方法！！！
>
> test4————》》参数：{"additionalProp1":{},"additionalProp3":{},"additionalProp2":{}}
>
> test4————》》joinPoint：execution(String ybplan.service.MyPlanService.test(JSONObject))

当我们切入的方法抛出异常时：

```java
 public String test(JSONObject json) {
        int a = 1 / 0;
        return "test";
    }
```

> 执行test方法！！！
>
> test4————》》参数：{"additionalProp1":{},"additionalProp3":{},"additionalProp2":{}}
>
> test4————》》joinPoint：execution(String ybplan.service.MyPlanService.test(JSONObject))

## 思考

当我们的切入方法上面加了事物的注解，自定义的aop是否会影响到事物？因为我们知道事物是通过aop实现的。