**处理 Exception 的几种实践**

> 参考文章：[处理 Exception 的几种实践，被很多团队采纳！](https://mp.weixin.qq.com/s?__biz=MzIyMDI5MzA3NQ==&mid=2247503097&idx=2&sn=6396b96165ab4f33e0ab3effcb6e9557&chksm=97ccbe45a0bb37531b10c87c7edcfaf7a599b3bb88eee052cca304c081daf5a19818fb6e3662&mpshare=1&scene=24&srcid=0619jLJia72qUY1mFGLameP8&sharer_sharetime=1624094366508&sharer_shareid=2d0b883b773a0513439aac3f9f6d55cd#rd)

​		在Java中处理异常并不是一个简单的事情。不仅仅初学者很难理解，即使一些有经验的开发者也需要花费很多时间来思考如何处理异常，包括需要处理哪些异常，怎样处理等等。

​		这也是绝大多数开发团队都会制定一些规则来规范对异常的处理的原因。而团队之间的这些规范往往是截然不同的。

​		本文给出几个被很多团队使用的异常处理最佳实践。

### 1. 在Finally块中清理资源或者使用try-with-resource语句

​		当使用类似InputStream这种需要使用后关闭的资源时，一个常见的错误就是在try块的最后关闭资源。

```java
public void doNotCloseResourceInTry() {
    FileInputStream inputStream = null;
    try {
        File file = new File("./tmp.txt");
        inputStream = new FileInputStream(file);
        // use the inputStream to read a file
        // do NOT do this
        inputStream.close();
    } catch (FileNotFoundException e) {
        log.error(e);
    } catch (IOException e) {
        log.error(e);
    }
}
```

​		上述代码在没有任何exception的时候运行是没有问题的。但是当try块中的语句抛出异常或者自己实现的代码抛出异常，那么就不会执行最后的关闭语句，从而资源也无法释放。

​		合理的做法则是将所有清理的代码都放到finally块中或者使用try-with-resource语句。

```java
public void closeResourceInFinally() {
    FileInputStream inputStream = null;
    try {
        File file = new File("./tmp.txt");
        inputStream = new FileInputStream(file);
        // use the inputStream to read a file
    } catch (FileNotFoundException e) {
        log.error(e);
    } finally {
        if (inputStream != null) {
            try {
                inputStream.close();
            } catch (IOException e) {
                log.error(e);
            }
        }
    }
}
public void automaticallyCloseResource() {
    File file = new File("./tmp.txt");
    try (FileInputStream inputStream = new FileInputStream(file);) {
        // use the inputStream to read a file
    } catch (FileNotFoundException e) {
        log.error(e);
    } catch (IOException e) {
        log.error(e);
    }
}
```

### 2. 指定具体的异常

​		尽可能的使用最具体的异常来声明方法，这样才能使得代码更容易理解。

```java
public void doNotDoThis() throws Exception {
    ...
}
public void doThis() throws NumberFormatException {
    ...

```

​		如上，NumberFormatException字面上即可以看出是数字格式化错误。

### 3. 对异常进行文档说明

​		当在方法上声明抛出异常时，也需要进行文档说明。和前面的一点一样，都是为了给调用者提供尽可能多的信息，从而可以更好地避免/处理异常。异常处理的 10 个最佳实践，这篇也推荐看下。

​		在Javadoc中加入throws声明，并且描述抛出异常的场景。

```java
/**
 * This method does something extremely useful ...
 *
 * @param input
 * @throws MyBusinessException if ... happens
 */
public void doSomething(String input) throws MyBusinessException {
    ...
}
```

### 4. 抛出异常的时候包含描述信息

​		在抛出异常时，需要尽可能精确地描述问题和相关信息，这样无论是打印到日志中还是监控工具中，都能够更容易被人阅读，从而可以更好地定位具体错误信息、错误的严重程度等。

​		但这里并不是说要对错误信息长篇大论，因为本来Exception的类名就能够反映错误的原因，因此只需要用一到两句话描述即可。

```java
try {
    new Long("xyz");
} catch (NumberFormatException e) {
    log.error(e);
}
```

​		NumberFormatException即告诉了这个异常是格式化错误，异常的额外信息只需要提供这个错误字符串即可。当异常的名称不够明显的时候，则需要提供尽可能具体的错误信息。

### 5. 首先捕获最具体的异常

​		现在很多IDE都能智能提示这个最佳实践，当你试图首先捕获最笼统的异常时，会提示不能达到的代码。当有多个catch块中，按照捕获顺序只有第一个匹配到的catch块才能执行。因此，如果先捕获IllegalArgumentException，那么则无法运行到对NumberFormatException的捕获。

```java
public void catchMostSpecificExceptionFirst() {
    try {
        doSomething("A message");
    } catch (NumberFormatException e) {
        log.error(e);
    } catch (IllegalArgumentException e) {
        log.error(e)
    }
}
```

### 6. 不要捕获Throwable

​		Throwable是所有异常和错误的父类。你可以在catch语句中捕获，但是永远不要这么做。如果catch了throwable，那么不仅仅会捕获所有exception，还会捕获error。而error是表明无法恢复的jvm错误。因此除非绝对肯定能够处理或者被要求处理error，不要捕获throwable。

```java
public void doNotCatchThrowable() {
    try {
        // do something
    } catch (Throwable t) {
        // don't do this!
    }
}
```

### 7. 不要忽略异常

​		很多时候，开发者很有自信不会抛出异常，因此写了一个catch块，但是没有做任何处理或者记录日志。

```java
public void doNotIgnoreExceptions() {
    try {
        // do something
    } catch (NumberFormatException e) {
        // this will never happen
    }
}
```

​		但现实是经常会出现无法预料的异常或者无法确定这里的代码未来是不是会改动(删除了阻止异常抛出的代码)，而此时由于异常被捕获，使得无法拿到足够的错误信息来定位问题。合理的做法是至少要记录异常的信息。

```java
public void logAnException() {
    try {
        // do something
    } catch (NumberFormatException e) {
        log.error("This should never happen: " + e);
    }
}
```

### 8. 不要记录并抛出异常

​		可以发现很多代码甚至类库中都会有捕获异常、记录日志并再次抛出的逻辑。如下：

```java
try {
    new Long("xyz");
} catch (NumberFormatException e) {
    log.error(e);
    throw e;
}
```

​		这个处理逻辑看着是合理的。但这经常会给同一个异常输出多条日志。如下：

```java
17:44:28,945 ERROR TestExceptionHandling:65 - java.lang.NumberFormatException: For input string: "xyz"
Exception in thread "main" java.lang.NumberFormatException: For input string: "xyz"
at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
at java.lang.Long.parseLong(Long.java:589)
at java.lang.Long.(Long.java:965)
at com.stackify.example.TestExceptionHandling.logAndThrowException(TestExceptionHandling.java:63)
at com.stackify.example.TestExceptionHandling.main(TestExceptionHandling.java:58)
```

​		如上所示，后面的日志也没有附加更有用的信息。如果想要提供更加有用的信息，那么可以将异常包装为自定义异常。

```java
public void wrapException(String input) throws MyBusinessException {
    try {
        // do something
    } catch (NumberFormatException e) {
        throw new MyBusinessException("A message that describes the error.", e);
    }
}
```

​		因此，仅仅当想要处理异常时才去捕获，否则只需要在方法签名中声明让调用者去处理。

### 9. 包装异常时不要抛弃原始的异常

​		捕获标准异常并包装为自定义异常是一个很常见的做法。这样可以添加更为具体的异常信息并能够做针对的异常处理。

​		需要注意的是，包装异常时，一定要把原始的异常设置为cause(Exception有构造方法可以传入cause)。否则，丢失了原始的异常信息会让错误的分析变得困难。

```java
public void wrapException(String input) throws MyBusinessException {
    try {
        // do something
    } catch (NumberFormatException e) {
        throw new MyBusinessException("A message that describes the error.", e);
    }
}
```

### 10. 不要使用异常控制程序的流程

​		不应该使用异常控制应用的执行流程，例如，本应该使用if语句进行条件判断的情况下，你却使用异常处理，这是非常不好的习惯，会严重影响应用的性能。

