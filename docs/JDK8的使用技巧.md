**JDK8的使用技巧**

##### flatMap的使用

###### **将二维集合转为一维集合**

```java
/**
 * @Author zjm
 * @Description: **将二维集合转为以为集合**
 * @Date: Created in 16:49 2021/6/16
 * @Modified By:
 */
public class FlatMap {
    public static void main(String[] args) {
        demo1();
    }
    public static void demo1(){
        List<List<Integer>> lists = new ArrayList<>();
        lists.add(Arrays.asList(1,2,3,4,5));
        lists.add(Arrays.asList(11,21,31,41,51));
        lists.add(Arrays.asList(12,22,32,42,52));
        lists.add(Arrays.asList(13,23,33,43,53));
        List<Integer> collect = lists.stream().flatMap(Collection::stream).collect(Collectors.toList());
        collect.forEach(System.out::println);
    }
}
```

