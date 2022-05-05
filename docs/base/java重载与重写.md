## java重载与重写



### 重载的定义

​		在Java语言中，要重载(Overload)一个方法，除了要与原方法具有相同的简单名称之外，还要求必须拥有一个与原方法不同的特征签名。

> 特征签名：Java代码的方法特征签名只包括方法名称、参数数量、参数顺序及参数类型

​		特征签名是指一个方法中各个参数在常量池中的字段符号引用的集合，也正是因为返回值不会包含在特征签名之中，所以Java语言里面是无法仅仅依靠返回值 的不同来对一个已有方法进行重载的。

**案例一：编译报错**`'test(int)' clashes with 'test(int)'; both methods have same erasure`

```java
		private long test(int a){
        return 1L;
    }

    private int test(int a){
        return 1;
    }
```

**案例二：编译通过**

```java
private long test(long a){
        return 1L;
    }

    private int test(int a){
        return 1;
    }
```

### 当重载遇上泛型（以下代码实在jdk6的基础上）

思考一下如下重载是否能够通过编译：

```java
public class TestClass {
    public void method(List<String> list) {
        System.out.println("invoke method(List<String> list)");
    }

    public void method(List<Integer> list) {
        System.out.println("invoke method(List<Integer> list)");
    }
}
```

​		当我们通过idea编写时，会提示如下错误

> `'method(List<String>)' clashes with 'method(List<Integer>)'; both methods have same erasure`

​		我们发现与**案例一**报了同样的错误，这不是和重载定义的特征签名相违背嘛？？？

​		答案是：这段代码是不能被编译的，因为参数`List <Integer>`和`List <String>`编译之后都被擦除了，变成了同一种的裸类型`List`，**类型擦除**导致这两个方法的特征签名变得一模一样。

我们将代码改为如下：

```java
public class TestClass {
    public String method(List<String> list) {
        System.out.println("invoke method(List<String> list)");
      return "";
    }

    public int method(List<Integer> list) {
        System.out.println("invoke method(List<Integer> list)");
      return 1;
    }
}
```

​		发现可以编译和执行成功，之所以这次能编译和执行成功，是因为两 个`met hod()`方法加入了不同的返回值后才能共存在一个`Class`文件之中，在Class文件格式之中， 只要描述符不是完全一致的两个方法就可以共存。也就是说，如果两个方法有相同的名称和特征签名，但返回值不同，那么也是可以合法共存于同一个Class文件中的。

[类型擦除]()



### 重写的定义