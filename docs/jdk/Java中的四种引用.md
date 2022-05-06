		我们知道判断一个对象是否还存活可以通过`引用计数算法`或者`可达性算法`进行判断，无论是通过引用计数算法判断对象的引用数量，还是通过可达性分析算法判断对象是否引用链可达，判定对象是否存活都和“引用”离不开关系。

​		为了高效的利用内存和回收内存，Java对引用的概念进行了扩充，将引用分为强引用(Strongly Re-ference)、软引用(Soft Reference)、弱引用(Weak Reference)和虚引用(Phantom Reference)4种，这4种引用强 度依次逐渐减弱。**在这种分类下，当内存空间还足够时，能保留在内存之中，如果内存空间在进行垃圾收集后仍然非常紧张，那就可以抛弃这些对象。**

#### 强引用

- 使用最普遍的引用。
- 强引用可以直接访问目标对象。
- 只要引用链没有断开，强引用就不会断开。- 当内存空间不足，抛出`OutOfMemoryError`终止程序也不会回收具有强引用的对象。
- 通过将对象设置为null来弱化引用,使其被回收
- 强引用可能导致内存泄漏。

```java
Object obj=new Object()
```

#### 软引用

- 对象处在有用但非必须的状态
- 只有当内存空间不足时, GC会回收该引用的对象的内存。
- 可以用来实现高速缓存（作用）--比如网页缓存、图片缓存

```java
/**
 * -Xmx10m
 * After gc, soft ref is exists
 * after create byte array ,soft ref is gc
 *
 */
public class SoftRef {
    public static class User{
        public User(int id,String name){
            this.id=id;
            this.name=name;
        }
        public int id;
        public String name;

        @Override
        public String toString(){
            return "[id="+String.valueOf(id)+",name="+name+"]";
        }
    }
    public static void main(String[] args) {
        User u=new User(1,"geym");
        SoftReference<User> userSoftRef = new SoftReference<User>(u);
        u=null;

        System.out.println(userSoftRef.get());
        System.gc();
        System.out.println("After GC:");
        System.out.println(userSoftRef.get());
				//这里的byte数组大小需要自己调到合适的大小，
        //如果不调试可能不会出现如下截图的结果
        //也可能在某些机器上会出现OOM
        byte[] b=new byte[1024*1084*10];
        System.gc();
        System.out.println(userSoftRef.get());
    }
}
```

运行结果

![image-20220506213010575](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205062130612.png)

​		每一个软引用都可以附带一个引用队列，当对象的可达性发生改变时 (**由可达变为不可达**)，软引用对象就会进入引用队列。通过这个引用队列，可以跟踪对象的回收情况。请看下面的代码:

```java
package interview.actual_combat;


import java.lang.ref.ReferenceQueue;
import java.lang.ref.SoftReference;

/**
 * -Xmx10m
 *
 */
public class SoftRefQ {
    public static class User{
        public User(int id,String name){
            this.id=id;
            this.name=name;
        }
        public int id;
        public String name;

        @Override
        public String toString(){
            return "[id="+String.valueOf(id)+",name="+name+"]";
        }
    }

    static ReferenceQueue<User> softQueue=null;
    public static class CheckRefQueue extends Thread{
        @Override
        public void run(){
14            while(true){
                if(softQueue!=null){
                    UserSoftReference obj=null;
                    try {
                        obj = (UserSoftReference) softQueue.remove();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    if(obj!=null)
                        System.out.println("user id "+obj.uid+" is delete");
24                }
            }
        }
    }

29    public static class UserSoftReference extends SoftReference<User>{
        int uid;
        public UserSoftReference(User referent, ReferenceQueue<? super User> q) {
            super(referent, q);
            uid=referent.id;
        }
35    }

    public static void main(String[] args) throws InterruptedException {
        Thread t=new CheckRefQueue();
        t.setDaemon(true);
        t.start();
        User u=new User(1,"geym");
        softQueue = new ReferenceQueue<User>();
43        UserSoftReference userSoftRef = new UserSoftReference(u,softQueue);
        u=null;
        System.out.println(userSoftRef.get());
        System.gc();
        //内存足够不会被回收
        System.out.println("After GC:");
        System.out.println(userSoftRef.get());

        System.out.println("try to create byte array and GC");
        byte[] b=new byte[1024*1084*10];
        System.gc();
        System.out.println(userSoftRef.get());
        Thread.sleep(1000);
    }
}
```

运行结果

![image-20220506214407281](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205062144324.png)

上述代码值得注意的地方有两个:

- 第29~35行实现了一个自定义的软引用类，扩展软引用的目的是记录`User.uid`，后续在引用队列中就可以通过这个`uid`字段知道哪个`User`实例被回收了。

- 第43行，在创建软引用时，指定了一个软引用队列，当给定的对象实例被回收 时，就会被加入这个引用队列，通过访问该队列可以跟踪对象的回收情况，代码第14~24 行就跟踪引用队列，打印对象的回收情况。

#### 弱引用

- 非必须的对象,比软引用更弱一些
- GC时会被回收
- 被回收的概率也不大,因为GC线程优先级比较低
- 适用于引用偶尔被使用且不影响垃圾收集的对象使用

```java
//WeakHashMap 里的entry可能会被GC自动删除，即使没有主动调用 remove() 或者 clear() 方法
Map<String, String> activeEngineResources = new WeakHashMap<>();
```

```java
import java.lang.ref.WeakReference;

public class WeakRef {
    public static class User{
        public User(int id,String name){
            this.id=id;
            this.name=name;
        }
        public int id;
        public String name;

        @Override
        public String toString(){
            return "[id="+String.valueOf(id)+",name="+name+"]";
        }
    }
    public static void main(String[] args) {
        User u=new User(1,"geym");//强引用
        WeakReference<User> userWeakRef = new WeakReference<User>(u);//弱引用
        u=null;//去掉强引用
        System.out.println(userWeakRef.get());
        System.gc();//GC
        //不管内存足够与否都会回收他们的内存
        System.out.println("After GC:");
        System.out.println(userWeakRef.get());
    }
}
```

运行结果

![image-20220506214833389](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205062148432.png)

#### 虚引用

​		虚引用也称为“幽灵引用”或者“幻影引用”，它是最弱的一种引用关系。一个对象是否有虚引用的 存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。**为一个对象设置虚 引用关联的唯一目的只是为了能在这个对象被收集器回收时收到一个系统通知。**

```java
package interview.actual_combat;



import java.lang.ref.PhantomReference;
import java.lang.ref.ReferenceQueue;

/**
 * finalize ÊÇ²»¿É¿¿µÄ
 * ÐéÒýÓÃÊÇ¿É¿¿µÄ
 * @author geym
 *
 */
public class TraceCanReliveObj {
    public static TraceCanReliveObj obj;
    static ReferenceQueue<TraceCanReliveObj> phantomQueue=null;
    public static class CheckRefQueue extends Thread{
        @Override
        public void run(){
            while(true){
                if(phantomQueue!=null){
                    PhantomReference<TraceCanReliveObj> objt=null;
                    try {
                        objt = (PhantomReference<TraceCanReliveObj>)phantomQueue.remove();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    if(objt!=null){
                        System.out.println("TraceCanReliveObj is delete by GC");
                    }
                }
            }
        }
    }

    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("CanReliveObj finalize called");
        obj=this;
    }
    @Override
    public String toString(){
        return "I am CanReliveObj";
    }
    public static void main(String[] args) throws InterruptedException{
        Thread t=new CheckRefQueue();
        t.setDaemon(true);
        t.start();

        phantomQueue = new ReferenceQueue<TraceCanReliveObj>();
40        obj=new TraceCanReliveObj();
        PhantomReference<TraceCanReliveObj> phantomRef = new PhantomReference<TraceCanReliveObj>(obj,phantomQueue);
42        obj=null;
43        System.gc();
        Thread.sleep(1000);
        if(obj==null){
            System.out.println("obj is null");
        }else{
            System.out.println("obj 可用");
        }
        System.out.println("第2次 gc");
        obj=null;
        System.gc();
        Thread.sleep(1000);
        if(obj==null){
            System.out.println("obj is null");
        }else{
            System.out.println("obj 可用");
        }
    }
}
```

​		上述代码中`TraceCanReliveObj`对象是一个在`finalize()`函数中可复活的对象。在代码第 40行，构造了`TraceCanReliveObj`对象的虚引用，并指定了引用队列。第42行将强引用去 除。第43行第一次进行GC，由于对象可复活，GC无法回收该对象。接着，在第52行进行 了第2次GC，由于`finalize()`只会被调用一次，因此第2次GC会回收对象，同时其引用队列 应该也会捕获到对象的回收。