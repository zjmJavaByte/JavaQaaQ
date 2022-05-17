# String为什么被设计成final

## HashMap

​		我们一个对象作为`HashMap`的key被放入`HashMap`之后，若该对象状态变化导致其`Hash Code`的变化，则会导致后面相同的的兑现作为`key`去`get`的时候无法获取关联的值。日常使用中，我们经常使用`String`对象作为`HashMap`的`key`。所以只有`String`对象为不可变对象的时候才能保证正确。







参考[final域的内存语义](https://github.com/zjmJavaByte/JavaQaaQ/blob/master/docs/%E5%A4%9A%E7%BA%BF%E7%A8%8B/final%E5%9F%9F%E7%9A%84%E5%86%85%E5%AD%98%E8%AF%AD%E4%B9%89.md)