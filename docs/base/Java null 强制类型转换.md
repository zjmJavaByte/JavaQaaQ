**Java null 强制类型转换**

### null强转给对象

#### 代码

```java
public class Cast {
    public static void main(String[] args) {
        Object obj = null;
        Integer s1 = (Integer)obj;
        System.out.println(s1);
    }
}
```

#### 测试结果

![image-20210725102641490](https://gitee.com/laoyouji1018/images/raw/master/img/20210725102641.png)

#### 结论

> 如果把null强转给对象，是不会抛异常的，本身对象是可以为null的

### Null强转为基本类型

#### 代码

```java
public class Cast {

    public static void main(String[] args) {
        Object obj = null;
        int s1 = (Integer)obj;
        System.out.println(s1);
    }

}
```

#### 测试结果

![image-20210725110542970](https://gitee.com/laoyouji1018/images/raw/master/img/20210725110543.png)

#### 结论

我们反编译后会发现调用了 intValue方法去获取value，所以抛出空指针错误：

```java
Object obj = null;
int s1 = ((Integer)obj).intValue();
System.out.println(s1);
```

### Null强转后的对象使用

#### 调用成员方法

##### 代码

```java
public class NULL {
    void say(int i){
        System.out.println("hello" + i);
    }
    public static void main(String[] args){
        NULL n = null;
        n.say(1);
    }
}

```

##### 测试结果

![image-20210725110948747](https://gitee.com/laoyouji1018/images/raw/master/img/20210725110948.png)

#### 调用static方法

##### 代码

```java
public class NULL {
    static void say(int i){
        System.out.println("hello" + i);
    }
    public static void main(String[] args){
        NULL n = null;
        n.say(1);
        ((NULL)null).say(2);
    }
}
```

##### 测试结果

![image-20210725111126305](https://gitee.com/laoyouji1018/images/raw/master/img/20210725111126.png)

##### 反编译class文件后

![image-20210725111236290](https://gitee.com/laoyouji1018/images/raw/master/img/20210725111236.png)

##### 结论

- 将null强制类型转换后得到的是一个空的该类型的引用变量
- 使用对象调用静态方法时，与该对象是否为空没有关系