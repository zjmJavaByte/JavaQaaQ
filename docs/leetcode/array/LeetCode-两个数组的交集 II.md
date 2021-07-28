**两个数组的交集 II**

### 题目

> 给定两个数组，编写一个函数来计算它们的交集。

###  示例

> 示例 1：
>
> 输入：nums1 = [1,2,2,1], nums2 = [2,2]
> 输出：[2,2]
> 示例 2:
>
> 输入：nums1 = [4,9,5], nums2 = [9,4,9,8,4]
> 输出：[4,9]

### 说明：

> 输出结果中每个元素出现的次数，应与元素在两个数组中出现次数的最小值一致。
>
> 我们可以不考虑输出结果的顺序。

### 解法

- 先对数组进行排序

先对两个数组进行排序，然后使用两个指针，分别指向两个数组开始的位置。

如果两个指针指向的`值相同`，说明这个值是他们的交集，就把这个值加入到集合list中，然后两个指针在分别往后移一步。

如果两个指针指向的`值不同`，那么指向的值`相对小的往后移一步，相对大的先不动`，然后再比较

一直重复上面的操作，直到其中一个指针不能再移动为止，最后再把集合list转化为数组即可。

```java
    public int[] intersect(int[] nums1, int[] nums2) {
        // 先对两个数组进行排序
        Arrays.sort(nums1);
        Arrays.sort(nums2);
        int i = 0;
        int j = 0;
        List<Integer> list = new ArrayList<>();
        while (i < nums1.length && j < nums2.length) {
            if (nums1[i] < nums2[j]) {
                // 如果i指向的值小于j指向的值，，说明i指向
                // 的值小了，i往后移一步
                i++;
            } else if (nums1[i] > nums2[j]) {
                // 如果i指向的值大于j指向的值，说明j指向的值
                // 小了，j往后移一步
                j++;
            } else {
                // 如果i和j指向的值相同，说明这两个值是重复的，
                // 把他加入到集合list中，然后i和j同时都往后移一步
                list.add(nums1[i]);
                i++;
                j++;
            }
        }
        //把list转化为数组
        int index = 0;
        int[] res = new int[list.size()];
        for (int k = 0; k < list.size(); k++) {
            res[index++] = list.get(k);
        }
        return res;
    }
```

- 使用map解决

还可以使用map来解决，具体操作如下

遍历`nums1`中的所有元素，把它存放到map中，其中key就是`nums1`中的元素，value就是这个元素在数组`nums1`中出现的次数。

遍历`nums2`中的所有元素，查看map中是否包含`nums2`的元素，如果包含，就把当前值加入到集合list中，然后对应的value要减1。

最后再把集合list转化为数组即可，代码如下

```java
    public int[] intersect(int[] nums1, int[] nums2) {
        HashMap<Integer, Integer> map = new HashMap<>();
        ArrayList<Integer> list = new ArrayList<>();

        //先把数组nums1的所有元素都存放到map中，其中key是数组中
        //的元素，value是这个元素出现在数组中的次数
        for (int i = 0; i < nums1.length; i++) {
            map.put(nums1[i], map.getOrDefault(nums1[i], 0) + 1);
        }

        //然后再遍历nums2数组，查看map中是否包含nums2的元素，如果包含，
        //就把当前值加入到集合list中，然后再把对应的value值减1。
        for (int i = 0; i < nums2.length; i++) {
            if (map.getOrDefault(nums2[i], 0) > 0) {
                list.add(nums2[i]);
                map.put(nums2[i], map.get(nums2[i]) - 1);
            }
        }

        //把集合list转化为数组
        int[] res = new int[list.size()];
        for (int i = 0; i < list.size(); i++) {
            res[i] = list.get(i);
        }
        return res;
    }
```



