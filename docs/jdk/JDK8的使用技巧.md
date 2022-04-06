**JDK8的使用技巧**

##### flatMap的使用

###### **将二维集合转为一维集合**

```java
/**
 * @Author zjm
 * @Description: 将二维集合转为一维集合
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

 

###### 使用递归生成父子结构

```java
 public Object treeList() {
        List<Corp> all = repository.findAll();
        List<Corp> result = all.stream()
                //corp.getLevel().equals(1) 意思是level为1的是顶级父级
                .filter(corp -> corp.getLevel().equals(1))
                .map(permission -> covert(permission, all)).collect(Collectors.toList());
        return result;
    }

    /**
     * 将权限转换为带有子级的权限对象
     * 当找不到子级权限的时候map操作不会再递归调用covert
     */
    private Corp covert(Corp corp, List<Corp> corpList) {
        Corp node = new Corp();
        BeanUtils.copyProperties(corp, node);
        List<Corp> children = corpList.stream()
                .filter(subCorp -> subCorp.getParentCode().equals(corp.getCode()))
                .map(subCorp -> covert(subCorp, corpList)).collect(Collectors.toList());
        node.setChildren(children);
        return node;
    }
```

