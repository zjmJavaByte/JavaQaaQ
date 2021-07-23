## 前言

​		虽然 `Object` 是一个具体的类，但它主要是为继承而设计的。它的所有非 `final` 方法（`equals、hashCode、toString、clone 和 finalize`）都有清晰的通用约定（` general contracts`），因为它们被设计为被子类重写。任何类要重写这些方法时，都有义务去遵从它们的通用约定；如果不这样做，将会阻止其他依赖于约定的类 (例如 `HashMap` 和 `HashSet`) 与此类一起正常工作。

## 覆盖 equals 方法时应遵守的约定

### 什么时候不需要覆盖

​		覆盖 equals 方法似乎很简单，但是有很多覆盖的方式会导致出错，而且后果可能非常严重。避免问题的最简单方法是不覆盖 equals 方法，在这种情况下，类的每个实例都只等于它自己。如果符合下列任何条件，就是正确的做法：

- **类的每个实例本质上都是唯一的。** 对于像 Thread 这样表示活动实体类而不是值类来说也是如此。Object 提供的 equals 实现对于这些类具有完全正确的行为。

```java
//活动实体类而不是值类：比较是不是同一个对象，而不去管属性值是不是一样；以下是Object中的equal方法
public boolean equals(Object obj) {
        return this == obj;
}
```

- **该类不需要提供「逻辑相等」测试。** 例如，`java.util.regex.Pattern` 可以覆盖 equals 来检查两个 Pattern 实例是否表示完全相同的正则表达式，但设计人员认为客户端不需要或不需要这个功能。在这种情况下，从 Object 继承的 equals 实现是理想的。

- **类是私有的或包私有的，并且你确信它的 equals 方法永远不会被调用。** 如果你非常想要规避风险，你可以覆盖 equals 方法，以确保它不会意外调用：

```java
@Override
public boolean equals(Object o) {
    throw new AssertionError(); // Method is never called
}
```

### 什么时候应该覆盖

