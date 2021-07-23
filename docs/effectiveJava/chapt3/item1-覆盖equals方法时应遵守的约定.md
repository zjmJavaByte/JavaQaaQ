## 前言

​		虽然 `Object` 是一个具体的类，但它主要是为继承而设计的。它的所有非 `final` 方法（`equals、hashCode、toString、clone 和 finalize`）都有清晰的通用约定（` general contracts`），因为它们被设计为被子类重写。任何类要重写这些方法时，都有义务去遵从它们的通用约定；如果不这样做，将会阻止其他依赖于约定的类 (例如 `HashMap` 和 `HashSet`) 与此类一起正常工作。

## 覆盖 equals 方法时应遵守的约定

### 什么时候不需要覆盖

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

- **最多只存在一个对象的类、枚举类型**

### 什么时候应该覆盖

​		当一个类有一个逻辑相等的概念，而这个概念不同于仅判断对象的同一性（相同对象的引用），并且超类还没有覆盖 equals。对于值类通常是这样。值类只是表示值的类，例如 Integer 或 String。使用 equals 方法比较引用和值对象的程序员希望发现它们在逻辑上是否等价，而不是它们是否引用相同的对象。

#### 覆盖equal方法遵守的约定

- 自反性：对于任何非空的引用值 x，`x.equals(x)` 必须返回 true。

- 对称性：对于任何非空引用值 x 和 y，当且仅当`x.equals(y)` 为true时， `y.equals(x)` 必须返回 true 。

违法上述约定的代码

```java
// Broken - violates symmetry!
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
}

// Broken - violates symmetry!
@Override
public boolean equals(Object o) {
    if (o instanceof CaseInsensitiveString)
    return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);

    if (o instanceof String) // One-way interoperability!
        return s.equalsIgnoreCase((String) o);

    return false;
    } ... // Remainder omitted
}

```

假设我们有一个不区分大小写的字符串和一个普通字符串

```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";
```

正如预期的那样，`cis.equals(s)` 返回 true。问题是，虽然 `CaseInsensitiveString `中的 `equals `方法知道普通字符串，但是 `String` 中的 `equals` 方法对不区分大小写的字符串不知情。因此，`s.equals(cis)` 返回 false，这明显违反了对称性。假设你将不区分大小写的字符串放入集合中：

```java
// Demonstration of the problem (Page 40)
    public static void main(String[] args) {
        CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
        String s = "polish";

        List<CaseInsensitiveString> list = new ArrayList<>();
        list.add(cis);

        System.out.println(list.contains(s));
    }
```

当前的` OpenJDK` 实现中`list.contains(s)`返回false

```java
false
```

修改equal方法

```java
    // Fixed equals method (Page 40)
@Override public boolean equals(Object o) {
      return o instanceof CaseInsensitiveString &&
          ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```

- 传递性：对于任何非空的引用值 x, y, z，如果 `x.equals(y)` 返回 true，`y.equals(z)` 返回 true，那么 `x.equals(z)` 必须返回 true。

让我们从一个简单的不可变二维整数点类开始：

```java
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point)o;
        return p.x == x && p.y == y;
    }
    ... // Remainder omitted
}

```

子类添加了一条影响 equals 比较的信息

```java
public class ColorPoint extends Point {
    private final Color color;//添加的字段属性

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
    ... // Remainder omitted
}

```

如果不覆盖父类Point 的equal方法

```java
public static void main(String[] args) {
        Point p = new Point(1, 2);
        ColorPoint cp = new ColorPoint(1, 2, Color.RED);
        System.out.println(p.equals(cp) + " " + cp.equals(p)); //比较的是x,y两个值是否相等
}
```

> true  true

如果覆盖父类Point 的equal方法，并且添加上color

```java

@Override
public boolean equals(Object o) {
    if (!(o instanceof ColorPoint))
        return false;
    return super.equals(o) && ((ColorPoint) o).color == color;
}
```

```java
public static void main(String[] args) {
        Point p = new Point(1, 2);
        ColorPoint cp = new ColorPoint(1, 2, Color.RED);
        System.out.println(p.equals(cp) + " " + cp.equals(p)); 
}
```

> true  false

从以上结果发现违反了对称性

为了达到对称性，让ColorPoint.equal()方法在实现混合比较时（父子类比较），忽略颜色

```java
//修改ColorPoint中的equal方法
@Override
public boolean equals(Object o) {
    if (!(o instanceof Point))
        return false;

    // If o is a normal Point, do a color-blind comparison
    if (!(o instanceof ColorPoint))
        return o.equals(this);

    // o is a ColorPoint; do a full comparison
    return super.equals(o) && ((ColorPoint) o).color == color;
}

```

```java
public static void main(String[] args) {
        // First equals function violates symmetry (Page 42)
        Point p = new Point(1, 2);
        ColorPoint cp = new ColorPoint(1, 2, Color.RED);
        System.out.println(p.equals(cp) + " " + cp.equals(p));

        // Second equals function violates transitivity (Page 42)
        ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
        Point p2 = new Point(1, 2);
        ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
        System.out.printf("%s %s %s%n",
                          p1.equals(p2), p2.equals(p3), p1.equals(p3));
    }
```

> true true
> true true false

虽然经过上面后保证了对称性，但是却违反了传递性

- 一致性：对于任何非空的引用值 x 和 y, `x.equals(y)` 的多次调用必须一致地返回 true 或一致地返回 false，前提是不修改 equals 中使用的信息。
