**存在重复元素**

### 题目

> 给定一个整数数组，判断是否存在重复元素。
>
> 如果存在一值在数组中出现至少两次，函数返回 true 。如果数组中每个元素都不相同，则返回 false 。

### 示例 1:

> 输入: [1,2,3,1]
> 输出: true

### 示例 2:

> 输入: [1,2,3,4]
> 输出: false

### 示例 3:

> 输入: [1,1,1,3,3,4,3,2,4,2]
> 输出: true

### 解答

- 先排序再比较

首先这题最容易想到的是暴力求解，但暴力求解效率很低，直接超时


```java
public boolean containsDuplicate(int[] nums) {
    for (int i = 0; i < nums.length; i++) {
        for (int j = i + 1; j < nums.length; j++) {
            if (nums[i] == nums[j]) {
                return true;
            }
        }
    }
    return false;
}
```
我们可以先对数组进行排序，然后再比较。因为排序之后如果有相同的，那么相同的值肯定是挨着的，我们只需要在排序之后两两比较即可。代码如下


```java
public boolean containsDuplicate(int[] nums) {
    Arrays.sort(nums);
    for (int ind = 1; ind < nums.length; ind++) {
        if (nums[ind] == nums[ind - 1]) {
            return true;
        }
    }
    return false;
}
```

- 
	使用set集合


我们知道set集合中的元素是不能有重复的，在添加的时候如果有重复的，会把之前的值给覆盖，并且返回false。我们遍历数组中的所有值，一个个添加到集合set中，在添加的时候如果返回false，就表示有重复的，直接返回true。


```java
public boolean containsDuplicate(int[] nums) {
    Set<Integer> set = new HashSet<>();
    for (int num : nums) {
        //因为集合set中不能有重复的元素，如果有重复的
        //元素添加，就会添加失败
        if (!set.add(num))
            return true;
    }
    return false;
}
```
当然还可以在换种写法，但效率没有上面的那种高


```java
public boolean containsDuplicate(int[] nums) {
    Set<Integer> set = new HashSet<>();
    for (int num : nums) {
        set.add(num);
    }
    //如果有重复的，set中会覆盖，导致size减小，
    //如果没有重复的，set的大小等于nums的长度
    return set.size() != nums.length;
}
```

