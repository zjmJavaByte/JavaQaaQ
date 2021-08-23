**始终覆盖 toString 方法**

## 前言

虽然 Object 提供 toString 方法的实现，但它返回的字符串通常不是类的用户希望看到的。

### 案例代码

```java
package effectivejava.chapter3.item12;

import java.math.BigInteger;
import java.util.ArrayList;

// Adding a toString method to PhoneNumber (page 52)
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "area code");
        this.prefix   = rangeCheck(prefix,   999, "prefix");
        this.lineNum  = rangeCheck(lineNum, 9999, "line num");
    }

    private static short rangeCheck(int val, int max, String arg) {
        if (val < 0 || val > max)
            throw new IllegalArgumentException(arg + ": " + val);
        return (short) val;
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof effectivejava.chapter3.item11.PhoneNumber))
            return false;
        PhoneNumber pn = (PhoneNumber)o;
        return pn.lineNum == lineNum && pn.prefix == prefix
                && pn.areaCode == areaCode;
    }

    @Override public int hashCode() {
        int result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        return result;
    }

    /**
     * Returns the string representation of this phone number.
     * The string consists of twelve characters whose format is
     * "XXX-YYY-ZZZZ", where XXX is the area code, YYY is the
     * prefix, and ZZZZ is the line number. Each of the capital
     * letters represents a single decimal digit.
     *
     * If any of the three parts of this phone number is too small
     * to fill up its field, the field is padded with leading zeros.
     * For example, if the value of the line number is 123, the last
     * four characters of the string representation will be "0123".
     */
//    @Override public String toString() {
//        return String.format("%03d-%03d-%04d",
//                areaCode, prefix, lineNum);
//    }

    public static void main(String[] args) {
        PhoneNumber jenny = new PhoneNumber(707, 867, 5309);
        System.out.println("Jenny's number: " + jenny);
    }
}

```

### 约定一

#### 定义

> **提供一个好的 toString 实现（能）使类更易于使用，使用该类的系统（也）更易于调试。**

- 实现tostring方法的打印

```java
 {Jenny=707-867-5309}
```

- 未实现tostring方法的打印

```java
{Jenny=PhoneNumber@163b91}
```

### 约定二

#### 定义

> **当实际使用时，toString 方法应该返回对象中包含的所有值得关注的信息**

#### 反例

```java
//此处并没有打印出真正需要的信息
Assertion failure: expected {abc, 123}, but was {abc, 123}.
```

### 约定三

#### 定义

> **无论你是否决定在文档中指定返回值的格式，你都应该清楚地记录你的意图。**

- 如果你指定格式

```java
/*
返回此电话号码的字符串表示形式。 该字符串由十二个字符组成，格式为“XXX-YYY-ZZZZ”，其中 XXX 为区号，YYY 为前缀，ZZZZ 为行号。 每个大写字母代表一个十进制数字。 如果此电话号码的三个部分中的任何一个太小而无法填充其字段，则该字段用前导零填充。 例如，如果行号的值为 123，则字符串表示的最后四个字符将为“0123”。
*/
@Override public String toString() {
        return String.format("%03d-%03d-%04d",
                areaCode, prefix, lineNum);
    }
```

- 如果你不指定格式

```java
/*
返回此方法的简要说明。表示的确切细节未指定并且可能会更改，但以下可能被视为典型：
"[Potion #9: type=love, smell=turpentine, look=india ink]"
*/
@Override
public String toString() { ... }
```

### 约定四

#### 定义

> 无论你是否指定了格式，都要 **提供对 toString 返回值中包含的所有信息提供一种可以通过编程访问的途径。**

#### 例外

- 在静态实用程序类中编写 toString 方法是没有意义的

- 在大多数 enum 类型中也不应该编写 toString 方法，因为 Java 为你提供了一个非常好的方法

#### 必须

- 在任何抽象类中编写 toString 方法，该类的子类共享公共的字符串表示形式
