### 加一

### 题目

> 给定一个由 整数 组成的 非空 数组所表示的非负整数，在该数的基础上加一。
>
> 最高位数字存放在数组的首位， 数组中每个元素只存储单个数字。
>
> 你可以假设除了整数 0 之外，这个整数不会以零开头。
>

### 示例

示例 1：

> 输入：digits = [1,2,3]
> 输出：[1,2,4]
> 解释：输入数组表示数字 123。
> 示例 2：

示例 2：

> 输入：digits = [4,3,2,1]
> 输出：[4,3,2,2]
> 解释：输入数组表示数字 4321。

示例 3：

> 输入：digits = [0]
> 输出：[1]

### 解答

```java
    public int[] plusOne(int[] digits) {
        int length = digits.length;
        for (int i = length - 1; i >= 0; i--) {
            if (digits[i] != 9) {
                //如果数组当前元素不等于9，直接加1
                //然后直接返回
                digits[i]++;
                return digits;
            } else {
                //如果数组当前元素等于9，那么加1之后
                //肯定会变为0，我们先让他变为0
                digits[i] = 0;
            }
        }
        //除非数组中的元素都是9，否则不会走到这一步，
        //如果数组的元素都是9，我们只需要把数组的长度
        //增加1，并且把数组的第一个元素置为1即可
        int temp[] = new int[length + 1];
        temp[0] = 1;
        return temp;
    }


	/*
	1、从最后一个值进行开始遍历
2、只要是%10==0，就直接继续遍历，否则返回值即可
3、最后，遍历完还没给出结果的，那肯定是 10，这种所以要往前加1
	*/
    public static int[] test2(int[] digits) {
        int length = digits.length;
        for (int i = length - 1; i >= 0; i--) {
            digits[i] = digits[i] + 1;//先在原基础上家1
            digits[i] = digits[i] % 10;//再跟10取余
            if (digits[i] != 0) {//如果取余后不等于0，那说明该数字不是9
                return digits;
            }
        }
        int[] ints = new int[length + 1];
        ints[0] = 1;
        return ints;
    }
```

