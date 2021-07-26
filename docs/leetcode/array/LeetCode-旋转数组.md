### 题目

> 给定一个数组，将数组中的元素向右移动 `k` 个位置，其中 `k` 是非负数。

### 示例 1:

> 输入: nums = [1,2,3,4,5,6,7], k = 3
> 输出: [5,6,7,1,2,3,4]
> 解释:
> 向右旋转 1 步: [7,1,2,3,4,5,6]
> 向右旋转 2 步: [6,7,1,2,3,4,5]
> 向右旋转 3 步: [5,6,7,1,2,3,4]

### 示例 2:

> 输入：nums = [-1,-100,3,99], k = 2
> 输出：[3,99,-1,-100]
> 解释: 
> 向右旋转 1 步: [99,-1,-100,3]
> 向右旋转 2 步: [3,99,-1,-100]

### 答案

- 使用临时数组

  ![](https://gitee.com/laoyouji1018/images/raw/master/img/20210725230945.png)

  > 可以使用一个临时数组，先把原数组的值存放到一个临时数组中，然后再把临时数组的值重新赋给原数组，重新赋值的时候要保证每个元素都要往后移k位，如果超过数组的长度就从头开始，所以这里可以使用(i + k) % length来计算重新赋值的元素下标

  ```java
    public void rotate(int nums[], int k) {
         int length = nums.length;
         int temp[] = new int[length];
         //把原数组值放到一个临时数组中，
         for (int i = 0; i < length; i++) {
             temp[i] = nums[i];
         }
         //然后在把临时数组的值重新放到原数组，并且往右移动k位
         for (int i = 0; i < length; i++) {
             nums[(i + k) % length] = temp[i];
         }
     }
  ```

- 多次反转

![](https://gitee.com/laoyouji1018/images/raw/master/img/20210725231004.png)

先反转全部数组，在反转前k个，最后在反转剩余的，如下所示


```java
public void rotate(int[] nums, int k) {
    int length = nums.length;
    k %= length;
    reverse(nums, 0, length - 1);//先反转全部的元素
    reverse(nums, 0, k - 1);//在反转前k个元素
    reverse(nums, k, length - 1);//接着反转剩余的
}

//把数组中从[start，end]之间的元素两两交换,也就是反转
public void reverse(int[] nums, int start, int end) {
    while (start < end) {
        int temp = nums[start];
        nums[start++] = nums[end];
        nums[end--] = temp;
    }
}
```


其实还可以在调整下，先反转前面的，接着反转后面的k个，最后在反转全部，原理都一样


```java
public void rotate(int[] nums, int k) {
    int length = nums.length;
    k %= length;
    reverse(nums, 0, length - k - 1);//先反转前面的
    reverse(nums, length - k, length - 1);//接着反转后面k个
    reverse(nums, 0, length - 1);//最后在反转全部的元素
}

//把数组中从[start，end]之间的元素两两交换,也就是反转
public void reverse(int[] nums, int start, int end) {
    while (start < end) {
        int temp = nums[start];
        nums[start++] = nums[end];
        nums[end--] = temp;
    }
}
```

