# Happens-Before

​		虽然虚拟机和执行系统会对指令进行一定的重排，但是为了保证程序的正确性，`JMM`通过`happens-before`规则将重排序分为以下两类定义：

- 会改变程序执行结果的重排序：对于会改变程序执行结果的重排序，`JMM`要求编译器和处理器必须禁止这种重排序。

- 不会改变程序执行结果的重排序：对于不会改变程序执行结果的重排序，`JMM`对编译器和处理器不做要求(`JMM`允许这种 重排序)。

以下罗列了一些`happens-before`基本原则，这些原则是指令重排不可违背的。

-  程序顺序原则:一个线程中的每个操作，`happens-before`于该线程中的任意后续操作。
-  `volatile`规则:`volatile`变量的写先发生于读，这保证了`volatile`变量的可见性。
-  监视器锁规则:解锁`(unlock)`必然发生在随后的加锁`(lock)`前。
-  传递性:A先于B，B先于C，那么A必然先于C。

![image-20220509154952522](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205091549567.png)

> - `1 happens-before 2和3 happens-before 4`由程序顺序规则产生。由于编译器和处理器都要 遵守`as-if-serial`语义(as-if-serial语义保证单线程内程序的执行结果不被改变)，也就是说，`as-if-serial`语义保证了程序顺序规则。因此，可以把程序顺序 规则看成是对`as-if-serial`语义的“封装”。
>
> - `2 happens-before 3`是由`volatile`规则产生。前面提到过，对一个`volatile`变量的读，总是能看到(任意线程)之前对这个`volatile`变量最后的写入。因此，`volatile`的这个特性可以保证实现` volatile`规则。
> - `1 happens-before 4`是由传递性规则产生的。这里的传递性是由`volatile`的内存屏障插入策略和`volatile`的编译器重排序规则共同来保证的。

-  start()规则：线程的`start()`方法先于它的每一个动作。
   -  假设线程A在执行的过程中，通过执行ThreadB.start()来启动线 程B;同时，假设线程A在执行ThreadB.start()之前修改了一些共享变量，线程B在开始执行后会 读这些共享变量。

![image-20220509155516110](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205091555138.png)

> ​		`1 happens-before 2`由程序顺序规则产生。`2 happens-before 4`由`start()`规则产 生。根据传递性，将有`1 happens-before 4`。这实意味着，线程A在执行`ThreadB.start()`之前对共享 变量所做的修改，接下来在线程`B`开始执行后都将确保对线程B可见。

-  join()规则：如果线程A执行操作`ThreadB.join()`并成功返回，那么线程B中的任意操作`happens-before`于线程A从`ThreadB.join()`操作成功返回。
   -  假设线程`A`在执行的过程中，通过执行`ThreadB.join()`来等待线 程B终止;同时，假设线程`B`在终止之前修改了一些共享变量，线程A从`ThreadB.join()`返回后会 读这些共享变量。

![image-20220509155500795](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205091555827.png)

> `2 happens-before 4`由`join()`规则产生;`4 happens-before 5`由程序顺序规则产生。 根据传递性规则，将有`2 happens-before 5`。这意味着，线程`A`执行操作`ThreadB.join()`并成功返 回后，线程`B`中的任意操作都将对线程`A`可见。

-  线程的中断(`interrupt()`)先于被中断线程的代码。
-  对象的构造函数执行结束先于`finalize()`方法。