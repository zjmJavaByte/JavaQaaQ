## JDK动态代理和cglib动态代理

#### 原理区别

- java动态代理是利用反射机制生成一个实现代理接口的匿名类，在调用具体方法前调用InvokeHandler来处理。


- 而cglib动态代理是利用asm开源包，把代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。

#### 应用目标

- 如果目标对象实现了接口，默认情况下会采用JDK的动态代理实现AOP 

- 如果目标对象实现了接口，可以强制使用CGLIB实现AOP 

- 如果目标对象没有实现了接口，必须采用CGLIB库，spring会自动在JDK动态代理和CGLIB之间转换

#### 两者的区别

- JDK动态代理只能对实现了接口的类生成代理，而不能针对类
- CGLIB是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法，因为是继承，所以该类或方法最好不要声明成final 

#### 代码实现

##### JDK动态代理

###### 定义接口

- 首先需要一个接口

```java
/**
 * @Author zjm
 * @Description:
 * @Date: Created in 11:22 2021/7/20
 * @Modified By:
 */
public interface Fruits {
    public void eat();
}
```

###### 实现类

- `Orange`实现类

```java
/**
 * @Author zjm
 * @Description:
 * @Date: Created in 11:22 2021/7/20
 * @Modified By:
 */
public class Orange implements Fruits{
    @Override
    public void eat(String str) {
        System.out.println("吃"+str+"橘子。。。");
    }
}
```

- `Banana`实现类

```java
/**
 * @Author zjm
 * @Description:
 * @Date: Created in 12:46 2021/7/20
 * @Modified By:
 */
public class Banana implements Fruits{
    @Override
    public void eat(String str) {
        System.out.println("吃"+str+"香蕉。。。");
    }
}
```

###### 代理类

- 代理工厂

```java
/**
 * @Author zjm
 * @Description:
 * @Date: Created in 11:23 2021/7/20
 * @Modified By:
 */
public class ProxyFactory {
    
    private Object target;
    
    //使用构造器给目标对象初始化
    public ProxyFactory(Object target) {
        this.target = target;
    }

    /*
    根据传入的目标对象，利用反射机制，返回一个代理对象，然后通过代理对象，调用目标对象方法
    */
    //给目标对象生成一个代理对象
    public Object getInstance(){
        return Proxy.newProxyInstance(target.getClass().getClassLoader(),
                target.getClass().getInterfaces(), (o, method, args) -> {
                    System.out.println("jdk代理开始");
                    //反射机制调用目标对象方法
                    Object invoke = method.invoke(target, args);
                    return invoke;
                });
    }
}
```

- Proxy.newProxyInstance源码

```java
//loader：指定当前对象使用的加载器，获取加载器的方法固定的
//interfaces：目标对象实现的接，使用泛型方法确认类型
//h：时间处理，执行目标对象的方法时，会触发事件处理方法，会把当前执行的目标对象方法作为参数传入
@CallerSensitive
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) {
    Objects.requireNonNull(h);
    Class<?> caller = System.getSecurityManager() == null ? null : Reflection.getCallerClass();
    Constructor<?> cons = getProxyConstructor(caller, loader, interfaces);
    return newProxyInstance(caller, cons, h);
}
```

###### 测试

```java
public class Test {

    public static void main(String[] args) {
        //创建目标对象
        Fruits target = new Orange();
        //给目标对象创建代理对象
        Fruits proxyInstance = (Fruits)new ProxyFactory(target).getInstance();
        proxyInstance.eat("三个");

        //创建目标对象
        Fruits target2 = new Banana();
        //创建代理对象
        Fruits proxyInstance2 = (Fruits)new ProxyFactory(target2).getInstance();
        proxyInstance2.eat("二个");
    }

}
```

###### 分析

上述代码的关键是`Proxy.newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler handler)`方法，该方法会根据指定的参数动态创建代理对象。三个参数的意义如下：

1. `loader`，指定代理对象的类加载器；
2. `interfaces`，代理对象需要实现的接口，可以同时指定多个接口；
3. `handler`，方法调用的实际处理者，代理对象的方法调用都会转发到这里（*注意1）。

`newProxyInstance()`会返回一个实现了指定接口的代理对象，对该对象的所有方法调用都会转发给`InvocationHandler.invoke()`方法。理解上述代码需要对Java反射机制有一定了解。动态代理神奇的地方就是：

1. 代理对象是在程序运行时产生的，而不是编译期；
2. **对代理对象的所有接口方法调用都会转发到`InvocationHandler.invoke()`方法**，在`invoke()`方法里我们可以加入任何逻辑，比如修改方法参数，加入日志功能、安全检查功能等；之后我们通过某种方式执行真正的方法体，示例中通过反射调用了Hello对象的相应方法，还可以通过RPC调用远程方法。

> 注意1：对于从Object中继承的方法，JDK Proxy会把`hashCode()`、`equals()`、`toString()`这三个非接口方法转发给`InvocationHandler`，其余的Object方法则不会转发。详见[JDK Proxy官方文档](https://docs.oracle.com/javase/7/docs/api/java/lang/reflect/Proxy.html)。

如果对JDK代理后的对象类型进行深挖，可以看到如下信息：

```shell
# Hello代理对象的类型信息
class=class jdkproxy.$Proxy0
superClass=class java.lang.reflect.Proxy
interfaces: 
interface jdkproxy.Hello
invocationHandler=jdkproxy.LogInvocationHandler@a09ee92
```

代理对象的类型是`jdkproxy.$Proxy0`，这是个动态生成的类型，类名是形如$ProxyN的形式；父类是`java.lang.reflect.Proxy`，所有的JDK动态代理都会继承这个类；同时实现了`Fruits`接口，也就是我们接口列表中指定的那些接口。

如果你还对`jdkproxy.$Proxy0`具体实现感兴趣，它大致长这个样子：

```java
// JDK代理类具体实现
public final class $Proxy0 extends Proxy implements Fruits
{
  ...
  public $Proxy0(InvocationHandler invocationhandler)
  {
    super(invocationhandler);
  }
  ...
  @Override
  public final String eat(String str){
    ...
    return super.h.invoke(this, m3, new Object[] {str});// 将方法调用转发给invocationhandler
    ...
  }
  ...
}
```

这些逻辑没什么复杂之处，但是他们是在运行时动态产生的，无需我们手动编写。

###### 优缺点

**优点：**

1. Java本身支持，不用担心依赖问题，随着版本稳定升级；
2. 代码实现简单；

**缺点：**

1. 目标类必须实现某个接口，换言之，没有实现接口的类是不能生成代理对象的；
2. 代理的方法必须都声明在接口中，否则，无法代理；
3. 执行速度性能相对cglib较低；

##### CGLib代理

###### 定义代理类

- 来看示例，假设我们有一个没有实现任何接口的类`HelloConcrete`：

```java
public class HelloConcrete {
	public String sayHello(String str) {
		return "HelloConcrete: " + str;
	}
}
```

###### 定义CGLibProxy

- 代理类，需要实现MethodInterceptor接口

```java
/**
 * @Author zjm
 * @Description:
 * @Date: Created in 13:47 2021/7/20
 * @Modified By:
 */
public class MyMethodInterceptor implements MethodInterceptor {

    private Object targetObject;// CGLib需要代理的目标对象

    public Object createProxyObject(Object obj) {
        this.targetObject = obj;
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(obj.getClass());
        enhancer.setCallback(this);
        Object proxyObj = enhancer.create();
        return proxyObj;// 返回代理对象
    }

    @Override
    public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        System.out.println("CGLib代理开始");
        Object invoke = method.invoke(targetObject, args);
        System.out.println("CGLib代理结束");
        return invoke;
    }
}
```

###### 测试

```java
public class Test {

    public static void main(String[] args) {

        HelloConcrete hello = (HelloConcrete) new CGLibProxy()
                .createProxyObject(new HelloConcrete());
        hello.sayHello("hello");
    }

}
```

###### 分析

​		上述代码中，我们通过CGLIB的`Enhancer`来指定要代理的目标对象、实际处理代理逻辑的对象，最终通过调用`create()`方法得到代理对象，**对这个对象所有非final方法的调用都会转发给`MethodInterceptor.intercept()`方法**，在`intercept()`方法里我们可以加入任何逻辑，比如修改方法参数，加入日志功能、安全检查功能等；通过调用`MethodProxy.invokeSuper()`方法，我们将调用转发给原始对象，具体到本例，就是`HelloConcrete`的具体方法。CGLIG中[MethodInterceptor](http://cglib.sourceforge.net/apidocs/net/sf/cglib/proxy/MethodInterceptor.html)的作用跟JDK代理中的`InvocationHandler`很类似，都是方法调用的中转站。

> 注意：对于从Object中继承的方法，CGLIB代理也会进行代理，如`hashCode()`、`equals()`、`toString()`等，但是`getClass()`、`wait()`等方法不会，因为它是final方法，CGLIB无法代理。

如果对CGLIB代理之后的对象类型进行深挖，可以看到如下信息：

```shell
# HelloConcrete代理对象的类型信息
class=class cglib.HelloConcrete$$EnhancerByCGLIB$$e3734e52
superClass=class lh.HelloConcrete
interfaces: 
interface net.sf.cglib.proxy.Factory
invocationHandler=not java proxy class
```

我们看到使用CGLIB代理之后的对象类型是`cglib.HelloConcrete$$EnhancerByCGLIB$$e3734e52`，这是CGLIB动态生成的类型；父类是`HelloConcrete`，印证了CGLIB是通过继承实现代理；同时实现了`net.sf.cglib.proxy.Factory`接口，这个接口是CGLIB自己加入的，包含一些工具方法。

注意，既然是继承就不得不考虑final的问题。我们知道final类型不能有子类，所以CGLIB不能代理final类型，遇到这种情况会抛出类似如下异常：

```shell
java.lang.IllegalArgumentException: Cannot subclass final class cglib.HelloConcrete
```

同样的，final方法是不能重载的，所以也不能通过CGLIB代理，遇到这种情况不会抛异常，而是会跳过final方法只代理其他方法。

如果你还对代理类`cglib.HelloConcrete$$EnhancerByCGLIB$$e3734e52`具体实现感兴趣，它大致长这个样子：

```java
// CGLIB代理类具体实现
public class HelloConcrete$$EnhancerByCGLIB$$e3734e52
  extends HelloConcrete
  implements Factory
{
  ...
  private MethodInterceptor CGLIB$CALLBACK_0; // ~~
  ...
  
  public final String sayHello(String paramString)
  {
    ...
    MethodInterceptor tmp17_14 = CGLIB$CALLBACK_0;
    if (tmp17_14 != null) {
	  // 将请求转发给MethodInterceptor.intercept()方法。
      return (String)tmp17_14.intercept(this, 
              CGLIB$sayHello$0$Method, 
              new Object[] { paramString }, 
              CGLIB$sayHello$0$Proxy);
    }
    return super.sayHello(paramString);
  }
  ...
}
```

上述代码我们看到，当调用代理对象的`sayHello()`方法时，首先会尝试转发给`MethodInterceptor.intercept()`方法，如果没有`MethodInterceptor`就执行父类的`sayHello()`。这些逻辑没什么复杂之处，但是他们是在运行时动态产生的，无需我们手动编写。

###### 优缺点

**优点：**

1. 代理的类无需实现接口；
2. 执行速度相对JDK动态代理较高；

**缺点：**

1. 字节码库需要进行更新以保证在新版java上能运行；
2. 动态创建代理对象的代价相对JDK动态代理较高；

**Tips：**

1. 代理的对象不能是final关键字修饰的