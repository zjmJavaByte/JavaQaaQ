**明智地覆盖 clone 方法**

## 前言

> Cloneable 接口的目的是作为对象的一个 mixin 接口，表面这样的对象允许被克隆
>
> **译注：mixin 是掺合，混合，糅合的意思，即可以将任意一个对象的全部或部分属性拷贝到另一个对象上。**

### 约定一

#### 定义

> 事实上，实现Cloneable接口的类是为了提供一个功能适当的共有clone方法

#### 分析

- 如果一个类实现了 Cloneable，对象的克隆方法返回对象的逐域拷贝；否则它会抛出 CloneNotSupportedException。

- 无需调用构造器就可以创建对象

对于任何对象 x，表达式

```java
x.clone() != x           true
```

对于任何对象 x，以下表达式可能为true，可能为false

```java
x.clone().getClass() == x.getClass()
```

### 约定二

#### 定义

> 如果一个类和它的所有超类（Object除外）都遵守这个约定，通过调用 super.clone 来获得

#### 分析

对于任何对象 x，表达式都为true

```java
x.clone().getClass() == x.getClass()
```

### 约定三

#### 定义

> **不可变类永远不应该提供克隆方法**，只会造成不必要的克隆浪费

### 约定四

#### 定义

> 如果类只包含基本类型的域，那仅仅需要调用super.clone()方法技能得到你想要的对象

#### 代码

```java
public final class PhoneNumber implements Cloneable {
    private final short areaCode, prefix, lineNum;
     @Override public PhoneNumber clone() {
        try {
            //能够强转的原因：Java 支持协变返回类型。换句话说，覆盖方法的返回类型可以是被覆盖方法的返回类型的子类
            return (PhoneNumber) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();  // Can't happen
        }
    }
}
```

### 约定五

#### 定义

> 如果类中包含一些引用类型域，必须确保clone的对象不会伤害到原始的对象

#### 反例

```java
package effectivejava.chapter3.item13;
import java.util.Arrays;

// A cloneable version of Stack (Pages 60-61)
public class Stack implements Cloneable {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    // Clone method for class with references to mutable state
    @Override public Stack clone() {
        try {
            //此处是直接调用的super.clone()方法
            return (Stack) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }

   
}

```

#### 正例

```java
//修改clone方法如下，对引用类型进行一次拷贝（也就是所熟知的深拷贝）
@Override public Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            result.elements = elements.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
```

#### 注意事项

> 如果`elements`域是final的，上述方案就不能正常工作，因为clone方法是被禁止给final域赋新值的，只是个根本问题：就像序列化一样，**Cloneable架构与引用可变对象的final域正常用法是不相兼容的**，除非在原始对象和克隆对象之间可以安全的共享此可变对象。为了使类称为可克隆的，可能有必要称为去掉某些域中的final修饰符。

