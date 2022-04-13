[TOC]

# HotSpot虚拟机对象

## 对象的创建

### 创建过程

1. 当Java虚拟机遇到一条字节码`new`指令时，首先将去检查这个指令的参数是否能在`常量池`中定位到一个类的`符号引用`，并且**检查这个符号引用代表的类是否已被加载、解析和初始化过**。如果没有，那必须先执行相应的类加载过程。

2. 在类加载检查通过后，接下来虚拟机将为新生**对象分配内存**。

   1. 对象所需内存的大小在类加载完成 后便可完全确定

   2. 为对象分配空间的任务实际上便等同于把一块确定 大小的内存块从Java堆中划分出来，**对象的内存分配方法**有如下两种：

      - **指针碰撞**：假设Java堆中内存是绝对规整的，所有被使用过的内存都被放在一 边，空闲的内存被放在另一边，中间放着一个指针作为分界点的指示器，那所分配内存就仅仅是把那 个指针向空闲空间方向挪动一段与对象大小相等的距离
      - **空闲列表**：如果Java堆中的内存并不是规整的，已被使用的内存和空闲的内存相互交错在一起，那 就没有办法简单地进行指针碰撞了，虚拟机就必须维护一个列表，记录上哪些内存块是可用的，在分 配的时候从列表中找到一块足够大的空间划分给对象实例，并更新列表上的记录

      > 选择哪种分配方式由Java堆是否规整决定，而Java堆是否规整又由所采用的垃圾收集器是否带有空间压缩整理(Compact)的能力决定

3. **对象创建在虚拟机中是非常频繁的行为，即使仅仅修改一个指针所指向的位置，在并发情况下也并不是线程安全的**，可能出现正在给对象 A分配内存，指针还没来得及修改，对象B又同时使用了原来的指针来分配内存的情况

   1. 一种是对分配内存空间的动作进行同步处理——实际上虚拟机是**采用`CAS`配上失败重试的方式**保证更新操作的原子性
   2. **另外一种是把内存分配的动作按照线程划分在不同的空间之中进行**，即每个线程在Java堆中预先分配一小块内存，称为本地线程分配缓冲(`Thread Local Allocation Buffer，TLAB`)，哪个线程要分配内存，就在哪个线程的本地缓冲区中分配，只有本地缓冲区用完 了，分配新的缓存区时才需要同步锁定。虚拟机是否使用TLAB，可以通过`-XX:+/-UseTLAB`参数来 设定。

4. **内存分配完成之后，虚拟机必须将分配到的内存空间(`但不包括对象头`)都初始化为零值**，如果使用了`TLAB`的话，这一项工作也可以提前至`TLAB`分配时顺便进行

5. 接下来，**Java虚拟机还要对对象进行必要的设置**，例如这个`对象是哪个类的实例`、`如何才能找到类的元数据信息`、`对象的哈希码`(实际上对象的哈希码会延后到真正调用Object ::hashCode()方法时才计算)、`对象的GC分代年龄`等信息。**这些信息存放在对象的对象头(Object Header)之中**

6. **在上面工作都完成之后，从虚拟机的视角来看，一个新的对象已经产生了。但是从Java程序的视 角看来，对象创建才刚刚开始**——构造函数，即Class文件中的·`<init>()`方法还没有执行，所有的字段都 为默认的零值，对象需要的其他资源和状态信息也还没有按照预定的意图构造好。**new指令之后会接着执行`<init> ()`方法，按照程序员的意愿对对象进行初始化**，这样一个真正可用的对象才算完全被构造出来。

```c++
// 确保常量池中存放的是已解释的类
        if (!constants -> tag_at(index).is_unresolved_klass()) {
            // 断言确保是klassOop和instanceKlassOop(这部分下一节介绍)
            oop entry = (klassOop) * constants -> obj_at_addr(index);
            assert (entry -> is_klass(),"Should be resolved klass");
            klassOop k_entry = (klassOop) entry;
            assert (k_entry -> klass_part()->oop_is_instance(), "Should be instanceKlass");
            instanceKlass * ik = (instanceKlass *) k_entry -> klass_part();
            // 确保对象所属类型已经经过初始化阶段
            if (ik -> is_initialized() && ik -> can_be_fastpath_allocated()) {
                retry:
                // 取对象长度
                size_t obj_size = ik -> size_helper();
                oop result = NULL;
                // 记录是否需要将对象所有字段置零值
                bool need_zero = !ZeroTLAB;
                // 是否在TLAB中分配对象
                if (UseTLAB) {
                    result = (oop) THREAD -> tlab().allocate(obj_size);
                }
                if (result == NULL) {
                    need_zero = true; // 直接在eden中分配对象
                    HeapWord * compare_to = *Universe::heap () -> top_addr();
                    HeapWord * new_top = compare_to + obj_size;
                    // cmpxchg是x86中的CAS指令，这里是一个C++方法，通过CAS方式分配空间，并发失败的话，转到retry中重试直至成功分配为止
                    if (new_top <= *Universe::heap () -> end_addr()){
                        if (Atomic::cmpxchg_ptr (new_top, Universe::heap() -> top_addr(), compare_to) !=compare_to)
                        { goto retry;
                        }
                        result = (oop) compare_to;
                    }
                }
                if (result != NULL) {
                    // 如果需要，为对象初始化零值
                    if (need_zero ) {
                        HeapWord * to_zero = (HeapWord *) result + sizeof(oopDesc) / oopSize;
                        obj_size -= sizeof(oopDesc) / oopSize;
                        if (obj_size > 0) {
                            memset(to_zero, 0, obj_size * HeapWordSize);
                        }
                    }
                    // 根据是否启用偏向锁，设置对象头信息 if (UseBiasedLocking) {
                    result -> set_mark(ik -> prototype_header());
                } else {
                    result -> set_mark(markOopDesc::prototype ());
                }
                result -> set_klass_gap(0);
                result -> set_klass(k_entry);
                // 将对象引用入栈，继续执行下一条指令 SET_STACK_OBJECT(result, 0);
                UPDATE_PC_AND_TOS_AND_CONTINUE(3, 1);
            }
        }
```



## 对象的内存布局

### 对象头(Header)

HotSpot虚拟机对象的对象头部分包括两类信息：

- **第一类是用于存储对象自身的运行时数据**，如哈希码(HashCode)、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等，官方称为`Mark Word`
- **对象头的另外一部分是类型指针**，即对象指向它的类型元数据的指针，Java虚拟机通过这个指针 来确定该对象是哪个类的实例
- 此外，**如果对象是一个Java数组**，那在对象头中还必须有一块用于记录数组长度的数据

![image-20220412214357306](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204122143338.png)

[JDK之JVM中Java对象的头部占多少byte](https://cloud.tencent.com/developer/article/1413543)

### 实例数据(Instance Data)

​		**接下来实例数据部分是对象真正存储的有效信息，即我们在程序代码里面所定义的各种类型的字 段内容，无论是从父类继承下来的，还是在子类中定义的字段都必须记录起来**.这部分的存储顺序会 受到虚拟机分配策略参数(`-XX:FieldsAllocationStyle`参数)和字段在Java源码中定义顺序的影响。 HotSpot虚拟机默认的分配顺序为`longs/doubles、ints、shorts/chars、bytes/booleans、oops(Ordinary Object Pointers，OOPs)`，从以上默认的分配策略中可以看到，相同宽度的字段总是被分配到一起存 放，在满足这个前提条件的情况下，在父类中定义的变量会出现在子类之前。如果HotSpot虚拟机的 `+XX:CompactFields`参数值为`true`(默认就为true)，那子类之中较窄的变量也允许插入父类变量的空 隙之中，以节省出一点点空间。

### 对齐填充(Padding)

​		对象的第三部分是对齐填充，这并不是必然存在的，也没有特别的含义，它仅仅起着占位符的作 用。由于HotSpot虚拟机的自动内存管理系统要求对象起始地址必须是8字节的整数倍，换句话说就是任何对象的大小都必须是8字节的整数倍。对象头部分已经被精心设计成正好是8字节的倍数(1倍或者 2倍)，因此，如果对象实例数据部分没有对齐的话，就需要通过对齐填充来补全。

## 对象的访问定位

​		创建对象自然是为了后续使用该对象，我们的Java程序会通过栈上的reference数据来操作堆上的具 体对象。

主流的访问方式主要有使用句柄和直接指针两种:

**句柄**

![image-20220412215033115](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204122150146.png)

- 如果使用句柄访问的话，Java堆中将可能会划分出一块内存来作为句柄池，reference中存储的就 是对象的句柄地址，而句柄中包含了对象实例数据与类型数据各自具体的地址信息

**直接指针**

![image-20220412215040739](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204122150778.png)

- 如果使用直接指针访问的话，Java堆中对象的内存布局就必须考虑如何放置访问类型数据的相关 信息，reference中存储的直接就是对象地址，如果只是访问对象本身的话，就不需要多一次间接访问 的开销，

> 就虚拟机HotSpot而言，它主要使用第二种方式进行对象访问(有例外情况，如果使用了`Shenandoah收集器`的 话也会有一次额外的转发，具体可参见第3章)，但从整个软件开发的范围来看，在各种语言、框架中 使用句柄来访问的情况也十分常见。