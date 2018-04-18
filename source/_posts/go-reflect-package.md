---
title: go reflect package
date: 2018-03-24 17:52:43
tags: [go,reflect]
categories: go
---

在学习nsq源码时，再次重温一下 go 的 reflect 包。

# variable of interface type 接口类型变量

> A variable of interface type stores a pair: the concrete value assigned to the variable, and that value's type descriptor. 

接口类型变量存着两部分，具体的值和值类型描述符。

```
var t *int = nil
var i interface{}
i = t
fmt.Println(i == nil) //false
```

# Anonymous

```
// A StructField describes a single field in a struct.
type StructField struct {
	// Name is the field name.
	Name string
	// PkgPath is the package path that qualifies a lower case (unexported)
	// field name. It is empty for upper case (exported) field names.
	// See https://golang.org/ref/spec#Uniqueness_of_identifiers
	PkgPath string

	Type      Type      // field type
	Tag       StructTag // field tag string
	Offset    uintptr   // offset within struct, in bytes
	Index     []int     // index sequence for Type.FieldByIndex
	Anonymous bool      // is an embedded field
}
```

可以通过 field 的 Anonymous 判断该字段是否是内嵌字段

# Elem vs Indirect

`reflect.Value` 是一个指针，那么 `v.Elem()`  和 `reflect.Indirect(v)` 等价的。
`reflect.Value` 不是指针，那么 `reflect.Indirect(v)`  返回自身。
`reflect.Value` 不是指针但是一个 `interface`  那么 `v.Elem()`  返回 `interface` 所包含的值
`reflect.Value` 不是指针也不是一个 `interface` 那么 `v.Elem()` 会发生panic。 

# reflect.ValueOf()
```
var x float64 = 3.4
v := reflect.ValueOf(x)
fmt.Println(v.CanSet()) //false 因为 v 是使用 x 的拷贝值创建的。
```

```
var x float64 = 3.4
v := reflect.ValueOf(&x)
fmt.Println(v.CanSet()) //flase 我们不需要修改指针本身，而是需要修改指针所指向的地方。
```

```
var x float64 = 3.4
v := reflect.ValueOf(&x).Elem()
fmt.Println(v.CanSet()) //true
```

反射是接口变量到反射对象
反射是反射对象到接口变量
要修改反射对象的值那么一定是要settable

# Reference
https://blog.golang.org/laws-of-reflection
