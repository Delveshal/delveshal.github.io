---
title: go slice 和 string
date: 2018-03-26 21:46:00
tags: [go,string,slice,reflect]
categories: [go]
---

# string
通过 reflect 包我们可以知道 string 的内部结构
```
type StringHeader struct {
	Data uintptr
	Len  int
}
```
`Data` 是指向底层数组的指针。
`Len` 是字符串的长度。
字符串是不可变的。

对一个字符串进行切割操作是不会发生底层数组拷贝，衍生字符串只是一个字符串的子串。（使用黑科技例外）
```
	s := "1,2,3,4"
	ss := strings.Split(s,",")
	for i,_ := range ss{
		t := (*reflect.StringHeader)(unsafe.Pointer(&ss[i]))
		fmt.Println(t.Data,ss[i])
	}
	sss := s[:3]
	fmt.Println((*reflect.StringHeader)(unsafe.Pointer(&sss)))
	fmt.Println((*reflect.StringHeader)(unsafe.Pointer(&s)))
	//4952506 1
	//4952508 2
	//4952510 3
	//4952512 4
	//&{4952506 3}
	//&{4952506 7}
```
# slice

```
type SliceHeader struct {
	Data uintptr
	Len  int
	Cap  int
}
```

`Data` 是指向底层数组的指针。
`Len` 是slice的长度。
`Cap` 是slice的容量。

```
func main() {

	sl := make([]int,0,10)
	sl = append(sl,1,2,3)
	sl2 := sl[:2]
	fmt.Println((*reflect.SliceHeader)(unsafe.Pointer(&sl)).Data)  //842350584048
	fmt.Println((*reflect.SliceHeader)(unsafe.Pointer(&sl2)).Data) //842350584048
	test(sl)
	a := (*reflect.SliceHeader)(unsafe.Pointer(&sl))
	fmt.Println(a) //&{842350584048 3 10}
	fmt.Println(sl) //[2 2 3]
	a.Len = 4
	fmt.Println(a) //&{842350584048 4 10}
	fmt.Println(sl) //[2 2 3 1]
}

func test(a []int){
	fmt.Println((*reflect.SliceHeader)(unsafe.Pointer(&a))) //&{842350584048 3 10}
	a[0] = 2
	a = append(a,1)
	fmt.Println((*reflect.SliceHeader)(unsafe.Pointer(&a))) //&{842350584048 4 10}
}
```

可以看出 `slice` 进行切割时，得出新的 `slice` 也是共享底层数据的。
以前看过一些文章说 `slice` 传参传的是引用，我认为不正确。其实也是值传递，传递时对 `slice` 结构体进行了拷贝。
从代码中可以看出，传 `slice` 参数进去后对 `slice` 进行了修改可能没有“影响”到原有 `slice`。
`a[0] = 2`和 `a = append(a,1)` 同时修改了底层数据，但是后者修改了结构体中的`Len`字段的值，而这个值只是一个拷贝值，对于原来的 `slice` 的 `Len` 没有进行修改。

我们要注意 `slice` 共享底层数据的特征。

# 官方栗子 译文

选取官方博客的一篇[文章](https://blog.golang.org/go-slices-usage-and-internals) “A possible "gotcha"”

正如前面说到的，一个重新切片的切片并没有对底层数据进行拷贝。一个完整的数组会被保存在内存直到没有任何引用。有时候这会使得程序保存着所有数据在内存就算只用到其中的一小部分。

举个栗子，这 `FindDigits` 函数加载了整个文件到内存并且寻找第一个连续的数字，返回一个新的切片。

```
var digitRegexp = regexp.MustCompile("[0-9]+")

func FindDigits(filename string) []byte {
    b, _ := ioutil.ReadFile(filename)
    return digitRegexp.Find(b)
}
```
这段代码像预期那样运行，但是返回的`[]byte` 指向的数组保存在整个文件。因为这个切片引用了原始数组，只要这个切片保存引用那么垃圾回收机制就不会释放原始数组。使用了很少的字节但是保存了整个文件到内存。
为了解决这个问题，我们可以拷贝我们感兴趣的数据到一个新的 `slice` 再返回。

```
func CopyDigits(filename string) []byte {
    b, _ := ioutil.ReadFile(filename)
    b = digitRegexp.Find(b)
    c := make([]byte, len(b))
    copy(c, b)
    return c
}
```
一个更加简洁的方法可以使用 `append`，这留给读者作为练习。

这样？
```
func CopyDigits(filename string) []byte {
	var digitRegexp = regexp.MustCompile("[0-9]+")
	b, _ := ioutil.ReadFile(filename)
	b = digitRegexp.Find(b)
	c := append([]byte{},b...)
	return c
}
```
