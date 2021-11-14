## 切片的容量

`[5]int` 是数组，而 `[]int` 是切片。二者看起来相似，实则是根本上不同的数据结构。

切片的数据结构中，包含一个指向数组的指针 `array` ，当前长度 `len` ，以及最大容量 `cap` 。在使用 `make([]int, len)` 创建切片时，实际上还有第三个可选参数 `cap` ，也即 `make([]int, len, cap)` 。在不声明 `cap` 的情况下，默认 `cap=len` 。当切片长度没有超过容量时，对切片新增数据，不会改变 `array` 指针的值。

当对切片进行 `append` 操作，导致长度超出容量时，就会创建新的数组，这会导致和原有切片的分离。在下例中

```go
a := make([]int, 5)
b := a[0:4]
a = append(a, 1)
a[1] = 5
fmt.Println(b)
// [0 0 0 0]
fmt.Println(a)
// [0 5 0 0 0 1]</pre>
```

由于 `a` 的长度超出了容量，所以切片 `a` 指向了一个增长后的新数组，而 `b` 仍然指向原来的老数组。所以之后对 `a` 进行的操作，对 `b` 不会产生影响。

试比较

```go
a := make([]int, 5, 6)
b := a[0:4]
a = append(a, 1)
a[1] = 5
fmt.Println(a, b)
// [0 5 0 0 0 1] [0 5 0 0]
```

本例中， `a` 的容量为6，因此在 `append` 后并未超出容量，所以 `array` 指针没有改变。因此，对 `a` 进行的操作，对 `b` 同样产生了影响。

## 扩容机制

下面看看用 `a := []int{}` 这种方式来创建切片会是什么情况。

```go
a := []int{}
for i := 0; i < 16; i++ {
    a = append(a, i)
    fmt.Print(cap(a), " ")
}
// 1 2 4 4 8 8 8 8 16 16 16 16 16 16 16 16

```

可以看到，空切片的容量为0，但后面向切片中添加元素时，并不是每次切片的容量都发生了变化。这是因为，如果增大容量，也即需要创建新数组，这时还需要将原数组中的所有元素复制到新数组中，开销很大，所以GoLang设计了一套扩容机制，以减少需要创建新数组的次数。但这导致无法很直接地判断 `append` 时是否创建了新数组。

如果一次添加多个元素，容量又会怎样变化呢？试比较下面两个例子：

```go
a := []int{}
for i := 0; i < 16; i++ {
    a = append(a, 1, 2, 3, 4, 5)
    fmt.Print(cap(a), " ")
}
// 6 12 24 24 48 48 48 48 48 96 96 96 96 96 96 96 </pre>

<pre class="cm-s-default" style="box-sizing: border-box; font-size: inherit; font-family: inherit; margin: 0px; overflow: visible; padding: 0px; border-radius: 0px; border-width: 0px; background: transparent; white-space: pre; overflow-wrap: normal; line-height: inherit; color: inherit; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;">a := []int{}
for i := 0; i < 16; i++ {
    a = append(a, 1, 2, 3, 4, 5, 6)
    fmt.Print(cap(a), " ")
}
// 6 12 24 24 48 48 48 48 96 96 96 96 96 96 96 96
```

那么，是不是说，当向一个空切片中插入 `2n-1` 个元素时，容量就会被设置为 `2n` 呢？我们来试试其他的数据类型。

```go
// int8
a := []int8{}
for i := 0; i < 16; i++ {
    a = append(a, 1, 2, 3, 4, 5, 6)
    fmt.Print(cap(a), " ")
}
// 8 16 32 32 32 64 64 64 64 64 128 128 128 128 128 128

// int16
fmt.Println()
b := []int16{}
for i := 0; i < 16; i++ {
    b = append(b, 1, 2, 3, 4, 5)
    fmt.Print(cap(b), " ")
}
// 8 16 32 32 32 64 64 64 64 64 128 128 128 128 128 128

// bool
fmt.Println()
c := []bool{}
for i := 0; i < 16; i++ {
    c = append(c, true, false, true, false, false)
    fmt.Print(cap(c), " ")
}
// 8 16 32 32 32 64 64 64 64 64 128 128 128 128 128 128

// float32
fmt.Println()
d := []float32{}
for i := 0; i < 16; i++ {
    d = append(d, 1.1, 2.2, 3.3, 4.4, 5.5)
    fmt.Print(cap(d), " ")
}
// 8 16 16 32 32 32 64 64 64 64 64 64 128 128 128 128 

// float64
fmt.Println()
e := []float64{}
for i := 0; i < 16; i++ {
    e = append(e, 1.1, 2.2, 3.3, 4.4, 5.5)
    fmt.Print(cap(e), " ")
}
// 6 12 24 24 48 48 48 48 48 96 96 96 96 96 96 96 

// string
fmt.Println()
f := []string{}
for i := 0; i < 16; i++ {
    f = append(f, "1.1", "2.2", "3.3", "4.4", "5.5")
    fmt.Print(cap(f), " ")
}
// 5 10 20 20 40 40 40 40 80 80 80 80 80 80 80 80 

// []int
fmt.Println()
g := [][]int{}
g1 := []int{1, 2, 3, 4, 5}
for i := 0; i < 16; i++ {
    g = append(g, g1, g1, g1, g1, g1)
    fmt.Print(cap(g), " ")
}
// 5 10 20 20 42 42 42 42 85 85 85 85 85 85 85 85

```

可以看到，根据切片对应数据类型的不同，容量增长的方式也有很大的区别。相关的源码包括：[src/runtime/msize.go](https://links.jianshu.com/go?to=https%3A%2F%2Fgolang.org%2Fsrc%2Fruntime%2Fmsize.go)，[src/runtime/mksizeclasses.go](https://links.jianshu.com/go?to=https%3A%2F%2Fgolang.org%2Fsrc%2Fruntime%2Fmksizeclasses.go)等。

我们再看看切片初始非空的情形。

```go
a := []int{1, 2, 3}
fmt.Println(cap(a))
// 3
for i := 0; i < 16; i++ {
    a = append(a, 1, 2)
    fmt.Print(cap(a), " ")
}
// 6 12 12 12 24 24 24 24 24 24 48 48 48 48 48 48
```

可以看到，与刚刚向空切片添加5个int的情况一致，向有3个int的切片中添加2个int，容量增长为6。

需要注意的是，`append` 对切片扩容时，如果容量超过了一定范围，处理策略又会有所不同。可以看看下面这个例子。

```go
a := []int{1, 2, 3, 4}
fmt.Println(cap(a))
// 4
for i := 0; i < 20; i++ {
    a = append(a, a...)
    fmt.Print(cap(a), " ")
}
// 8 16 32 64 128 256 512 1024 2560 5120 10240 20480 40960 80896 158720 310272 606208 1184768 2314240 4520960
```

具体为什么会是这样的变化过程，还需要从[源码](https://links.jianshu.com/go?to=https%3A%2F%2Fgolang.org%2Fsrc%2Fruntime%2Fslice.go)中寻找答案。下面是 `src/runtime/slice.go` 中的 `growslice` 函数中的核心部分。

```go
// src/runtime/slice.go
func growslice(et *_type, old slice, cap int) slice {
// ...省略部分
    newcap := old.cap
    doublecap := newcap + newcap
    if cap > doublecap {
        newcap = cap
    } else {
        if old.len < 1024 {
            newcap = doublecap
        } else {
            // Check 0 < newcap to detect overflow
            // and prevent an infinite loop.
            for 0 < newcap && newcap < cap {
                newcap += newcap / 4
            }
            // Set newcap to the requested cap when
            // the newcap calculation overflowed.
            if newcap <= 0 {
                newcap = cap
            }
        }
    }
// ...省略部分
}
```

- 当需要的容量超过原切片容量的两倍时，会使用需要的容量作为新容量。
- 当原切片长度小于1024时，新切片的容量会直接翻倍。而当原切片的容量大于等于1024时，会反复地增加25%，直到新容量超过所需要的容量。

### 结论

GoLang中的切片扩容机制，与切片的数据类型、原本切片的容量、所需要的容量都有关系，比较复杂。对于常见数据类型，在元素数量较少时，大致可以认为扩容是按照翻倍进行的。但具体情况需要具体分析。

> 为了避免因为切片是否发生扩容的问题导致bug，最好的处理办法还是在必要时使用 `copy` 来复制数据，保证得到一个新的切片，以避免后续操作带来预料之外的副作用。



作者：吴子寒
链接：https://www.jianshu.com/p/54be5b08a21c