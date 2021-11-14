# Java接口成员变量和方法默认修饰符

### 成员修饰符 

Java的interface中，成员变量的默认修饰符为：public static final

所以我们在interface中定义成员变量的时候，可以

1：public static final String name = "张三";

2：String name = "张三";

以上两种都可以，老司机一般都是第二种。既然是静态最终的变量，也就意味着在外面访问的时候不能修改这个成员变量的值。所以在接口中定义成员变量的，一般都是常量。不会修改的。如果要进行修改的话，定义在接口具体实现类中。

### 方法修饰符

说完成员变量的默认修饰符，顺便也提下方法的默认修饰符，方法的默认修饰符是：public abstract

即：公共抽象的，就是用来被实现该接口的类去实现该方法。所以在接口中定义方法时候，也有两种方式

1：public abstract List<String> getUserNames(Long companyId);

2：List<String> getUserNames(Long companyId);

同样老司机都是第二种。

接口只是对一类事物属性和行为的更高次抽象；对修改关闭，对扩展开放，可以说是java中开闭原则的一种体现吧。

### 里氏替换原则

> 如果方法覆盖了超类中的一个方法，子类中的访问级别**不允许低于**超类中的访问级别

接口就是这条原则的一种特例

