## 前言

​		**在覆盖 equals 的类中，必须覆盖 hashCode。** 如果你没有这样做，你的类将违反 hashCode 的一般约定，这将阻止该类在 HashMap 和 HashSet 等集合中正常运行。

## Object规范的约定

- 当在应用程序执行期间对对象重复调用 hashCode 方法时，它必须一致地返回相同的值，前提是不对 equals 比较中使用的信息进行修改。在不同的应用程序执行过程中，执行hashCode方法返回的值可以不同。



- 如果根据 `equals(Object)` 方法判断出两个对象是相等的，那么在两个对象上调用 hashCode 必须产生相同的整数结果。

如果违反上述规定：

```java
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = areaCode;
        this.prefix   = prefix;
        this.lineNum  = lineNum;
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof PhoneNumber))
            return false;
        PhoneNumber pn = (PhoneNumber)o;
        return pn.lineNum == lineNum && pn.prefix == prefix
                && pn.areaCode == areaCode;
    }

    //此处并没有重写hashCode方法
    // Remainder omitted - note that hashCode is REQUIRED (Item 11)!
}
```



```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "Jenny");
m.get(new PhoneNumber(707, 867, 5309));
```

返回值

> null

结论

> PhoneNumber 类未能覆盖 hashCode，导致两个相等的实例具有不相等的 hash 代码，这违反了 hashCode 约定。因此，get 方法查找PhoneNumber的 hash 桶可能会与 put 方法存储PhoneNumber的 hash 桶不同。即使这两个实例碰巧 hash 到同一个 hash 桶上，get 方法也必定会返回 null，因为 HashMap 有一个优化，它缓存与每个条目相关联的 hash 码，如果 hash 码不匹配，就不会检查对象是否相等。

缓存与每个条目相关联的 hash 码

![image-20210729130840441](https://gitee.com/laoyouji1018/images/raw/master/img/20210729130841.png)

- 如果根据 `equals(Object)` 方法判断出两个对象不相等，则不一定要求在每个对象上调用 hashCode 时必须产生不同的结果。但是，程序员应该知道，为不相等的对象生成不同的结果可能会提高散列表的性能。

