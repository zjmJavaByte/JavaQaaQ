**明智地覆盖 clone 方法**

## 前言

> Cloneable 接口的目的是作为对象的一个 mixin 接口，表明这样的对象允许被克隆
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

#### 特例

> **仅仅递归地调用clone并不总是足够的。**例如，假设你正在为 hash 表编写一个克隆方法， hash 表的内部由一组 bucket 组成，每个 bucket 键-值对的引用都指向原先的链表。

```java
public class HashTable implements Cloneable {
    private Entry[] buckets = ...;

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
    } ... // Remainder omitted
}
```

假设你只是递归地克隆 bucket 数组，就像我们对 Stack 所做的那样：

```java
// Broken clone method - results in shared mutable state!
@Override
public HashTable clone() {
    try {
        HashTable result = (HashTable) super.clone();
        result.buckets = buckets.clone();
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

尽管克隆具有自己的 bucket 数组，但该数组引用的链接列表与原始链表相同，这很容易导致克隆和原始的不确定性行为。要解决这个问题，你必须复制包含每个 bucket 的链表。这里有一个常见的方法：

```java
// Recursive clone method for class with complex mutable state
public class HashTable implements Cloneable {

private Entry[] buckets = ...;

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }

        // Recursively copy the linked list headed by this Entry
        Entry deepCopy() {
            return new Entry(key, value,next == null ? null : next.deepCopy());
        }
    }

    @Override
    public HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];

            for (int i = 0; i < buckets.length; i++)
                if (buckets[i] != null)
                    result.buckets[i] = buckets[i].deepCopy();

            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    } ... // Remainder omitted
}

```

虽然这种技术很可爱，而且如果 bucket 不太长也可以很好地工作，但是克隆链表并不是一个好方法，因为它为链表中的每个元素消耗一个堆栈帧。如果列表很长，很容易导致堆栈溢出。为了防止这种情况的发生，你可以用迭代替换 deepCopy 中的递归：

```java
// Iteratively copy the linked list headed by this Entry
Entry deepCopy() {
    Entry result = new Entry(key, value, next);
    for (Entry p = result; p.next != null; p = p.next)
        p.next = new Entry(p.next.key, p.next.value, p.next.next);
    return result;
}
```

#### hashMap中的clone

> 克隆复杂可变对象的最后一种方法是调用 super.clone，将结果对象中的所有字段设置为初始状态，然后调用更高级别的方法重新生成原始对象的状态。

```java
public Object clone() {
        HashMap result;
        try {
            //调用 super.clo
            result = (HashMap)super.clone();
        } catch (CloneNotSupportedException var3) {
            throw new InternalError(var3);
        }
        //设置为初始状态
        result.reinitialize();
    	//调用更高级别的方法重新生成原始对象的状态
        result.putMapEntries(this, false);
        return result;
    }
```

调用更高级别的方法重新生成原始对象的状态

```java
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
        int s = m.size();
        if (s > 0) {
            if (this.table == null) {
                float ft = (float)s / this.loadFactor + 1.0F;
                int t = ft < 1.07374182E9F ? (int)ft : 1073741824;
                if (t > this.threshold) {
                    this.threshold = tableSizeFor(t);
                }
            } else if (s > this.threshold) {
                this.resize();
            }

            Iterator var8 = m.entrySet().iterator();

            while(var8.hasNext()) {
                Entry<? extends K, ? extends V> e = (Entry)var8.next();
                K key = e.getKey();
                V value = e.getValue();
                this.putVal(hash(key), key, value, false, evict);
            }
        }
    }
```

##### 注意事项

> **与构造函数一样，clone方法决不能在构建的过程中，调用可覆盖方法。**如果 clone 调用一个在子类中被重写的方法，这个方法将在克隆中的状态之前执行，很可能导致克隆对象和原始对象的之间不一致问题。因此，前一段中讨论的 put(key, value)方法应该是 final 或 private 方法。

