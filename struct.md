# go struct
## 2014-07-06



## 目录
* make vs new
* struct 成员函数初始化方法
* 扩展、继承



## make vs new
* [make 的官方文档](http://golang.org/pkg/builtin/#make)明确说明了: 
 * Unlike new, make's return type is the same as the type of its argument, not a pointer to it. 
* [new 的官方文档](http://golang.org/pkg/builtin/#new)明确说明了: 
 * The new built-in function allocates memory. The first argument is a type, not a value, and the value returned is a pointer to a newly allocated zero value of that type.
* 所以 make 和 new 的区别就是:
 * make 返回的是一个 Type 类型的变量
 * new 返回的是一个 Type 类型的指针


### 实现一个简单的 new 函数
```go
 func newInt() *int {
   var i int
   return &i
 }
 someVar := newInt()
```
* 当然 map, slice, array 等都可以通过 make 实现一个 new 函数


### struct 的初始化方法
* new(Type)
* make(Type)
* Type{} like make
* &Type{} like new



### struct 成员函数初始化方法
```go
 package main
 
 import "fmt"
 
 type Rectangle struct {
     length, width int
 }
 
 func (r Rectangle) Area() int {
     return r.length * r.width
 }
 func main() {
     r1 := Rectangle{4, 3}
     fmt.Println("Rectangle is: ", r1)
     fmt.Println("Rectangle area is: ", r1.Area())
 }
 // Rectangle is: {4 3}
 // Rectangle area is: 12
```


### struct 成员函数初始化方法
* 指针和非指针两种方式

```go
 package main
 
 import "fmt"
 
 type Rectangle struct {
     length, width int
 }
 
 func (r Rectangle) Area_by_value() int {
     return r.length * r.width
 }
 
 func (r *Rectangle) Area_by_reference() int {
     return r.length * r.width
 }
 
 func main() {
     r1 := Rectangle{4, 3}
     fmt.Println("Rectangle is: ", r1)
     fmt.Println("Rectangle area is: ", r1.Area_by_value())
     fmt.Println("Rectangle area is: ", r1.Area_by_reference())
     fmt.Println("Rectangle area is: ", (&r1).Area_by_value())
     fmt.Println("Rectangle area is: ", (&r1).Area_by_reference())
 }

 // Rectangle is: {4 3}
 // Rectangle area is: 12
 // Rectangle area is: 12
 // Rectangle area is: 12
 // Rectangle area is: 12
```
Note:
 这么容易就可以给 struct 添加成员函数, 是不是以为着咱们可以他 time package 添加一个函数呢


### 指针和非指针方式真的一样吗?
```go
 package main
 
 import "fmt"
 
 type Rectangle struct {
     length, width int
     val, val_by_ref int
 }
 
 func (r Rectangle) String() string{
     str := fmt.Sprintf("length: %d, whdth: %d, val: %d, val_by_ref: %d", r.length, r.width, r.val, r.val_by_ref)
     return str
 }
 func (r Rectangle) Area_by_value() {
     r.val = r.length * r.width
 }
 
 func (r *Rectangle) Area_by_reference() {
     r.val_by_ref = r.length * r.width
 }
 
 func main() {
     r1 := Rectangle{length:4, width:3}
     fmt.Println("Rectangle is: ", r1)
 
     r1.Area_by_value()
     r1.Area_by_reference()
 
     fmt.Println("Rectangle area is: ", r1.val)
     fmt.Println("Rectangle area is: ", r1.val_by_ref)
 }

 // Rectangle is:  length: 4, whdth: 3, val: 0, val_by_ref: 0
 // Rectangle area is:  0
 // Rectangle area is:  12
```


### why?
```go
 package main
 
 import "fmt"
 
 type Rectangle struct {
     length, width int
     val, val_by_ref int
 }
 
 func (r Rectangle) String() string{
     str := fmt.Sprintf("length: %d, whdth: %d, val: %d, val_by_ref: %d", r.length, r.width, r.val, r.val_by_ref)
     return str
 }
 func (r Rectangle) ref() {
     fmt.Printf("obj ref ref is: %p\n", &r)
 }
 
 func (r *Rectangle) ref_by_reference() {
     fmt.Printf("obj ref ref_by_reference is: %p\n", r)
 }
 
 func main() {
     r1 := new(Rectangle)
     fmt.Printf("obj ref is: %p\n", r1)
     r1.ref()
     r1.ref_by_reference()
 
 }
 // obj ref is: 0xc210049000
 // obj ref ref is: 0xc210049040
 // obj ref ref_by_reference is: 0xc210049000
```
* 是指针时指向的是同一个 struct 对象
* 非指针时复制了一个 struct 对象


### 给 time package 添加一个成员函数
* 现在我们已经知道如何添加成员函数了, 现在我们为 time package 添加一个成员函数

```go
 func (t time.Time) first5Chars() string {
     return time.LocalTime().String()[0:5]
 }
 // cannot define new methods on non-local type time.Time
```
* 因为不是在同一个 package 文件中声明的, 所以不能添加成员函数
* 那么我们是否有办法扩展 time package 的功能呢?



### 扩展、继承
```go
 package main
 
 import "fmt"
 import "time"
 
 type myTime struct {
     time.Time //anonymous field
 }
 
 func (t myTime) first5Chars() string {
     return t.Time.String()[0:5]
 }
 
 func main() {
     m := myTime{*time.LocalTime()} //since time.LocalTime returns an address, we convert it to a value with *
     fmt.Println("Full time now:", m.String()) //calling existing String method on anonymous Time field
     fmt.Println("First 5 chars:", m.first5Chars()) //calling myTime.first5Chars
 }
 // Full time now: Tue Nov 10 23:00:00 UTC 2009
 // First 5 chars: Tue N
```


### 扩展、继承
* beengo 也是采用同样的方式实现"继承"的
```go
 // github.com/astaxie/beego/config/json.go
 43 // A Config represents the json configuration.
 44 // Only when get value, support key as section:name type.
 45 type JsonConfigContainer struct {
 46 	data map[string]interface{}
 47 	sync.RWMutex
 48 }

 125 // WriteValue writes a new value for key.
 126 func (c *JsonConfigContainer) Set(key, val string) error {
 127 	c.Lock()
 128 	defer c.Unlock()
 129 	c.data[key] = val
 130 	return nil
 131 }
```


### 扩展、继承
```go
 package main
 
 import (
     "fmt"
 )
 
 type Mammal struct {
     Lung string
 }
 
 type Fish struct {
     Gills string
 }
 
 type Amphibian struct {
     Mammal
     Fish
     Name string
 }
 
 func main() {
     salamander := new(Amphibian)
     salamander.Name = "salamander"
     fmt.Print(salamander.Name + " ")
 
     salamander.Lung = "Lung"
     fmt.Print("has " + salamander.Lung)
 
     salamander.Gills = "Gills"
     fmt.Println(" and " + salamander.Gills)
 }

 // salamander has Lung and Gills
```


### 多态性
```go
 package main
 
 import (
     "fmt"
 )
 
 type Mammal struct {
     Lung string
 }
 
 type Fish struct {
     Gills string
 }
 
 type Amphibian struct {
     Mammal
     Fish
     Name string
 }
 
 func main() {
     salamander := Amphibian{}
     salamander.Name = "salamander"
     fmt.Print(salamander.Name + " ")
 
     salamander.Lung = "Lung"
     fmt.Print("has " + salamander.Lung)
 
     salamander.Gills = "Gills"
     fmt.Println(" and " + salamander.Gills)
 
     nick := salamander.(Mammal)
     fmt.Println("Nick has a " + nick.Lung)
 }

 // ./struct-extend-2.go:32: invalid type assertion: salamander.(Mammal) (non-interface type Amphibian on left)
```
* 错误的使用方法


### 多态性
* 正确的使用方法

```go
 package main
 
 import (
     "fmt"
 )
 
 type Mammal struct {
     Lung string
 }
 
 type Fish struct {
     Gills string
 }
 
 type Amphibian struct {
     Mammal
     Fish
     Name string
 }
 
 func main() {
     salamandar := Amphibian{}
     salamandar.Name = "Salamandar"
     fmt.Print(salamandar.Name + " ")
 
     salamandar.Lung = "Lung"
     fmt.Print("has " + salamandar.Lung)
 
     salamandar.Gills = "Gills"
     fmt.Println(" and " + salamandar.Gills)
 
     lion := salamandar.Mammal
     fmt.Println("Lion has a " + lion.Lung)
 }

 // Salamandar has Lung and Gills
 // Lion has a Lung
```


### field 冲突
```go
 package main
 
 import (
     "fmt"
 )
 
 type Mammal struct {
     Lung string
     Lips string
 }
 
 type Fish struct {
     Gills string
     Lips string
 }
 
 type Amphibian struct {
     Mammal
     Fish
     Name string
 }
 
 func main() {
     salamander := new(Amphibian)
     salamander.Name = "salamander"
     fmt.Print(salamander.Name + " ")
 
     salamander.Lung = "Lung"
     fmt.Print("has " + salamander.Lung)
 
     salamander.Gills = "Gills"
     fmt.Println(" and " + salamander.Gills)
 
     salamander.Lips = "lips"
     mt.Println(salamander.Name + " has " + salamander.Lips)
 }
 // ./mulity-extend.go:34: ambiguous selector salamander.Lips
```

* 多重扩展的时候如果不同的基类有相同的 field 的时候直接引用就会报错, 因为子 struct 不知道你想引用的是哪个基类的 field


### field 冲突
```go
 package main
 
 import (
     "fmt"
 )
 
 type Mammal struct {
     Lung string
     Lips string
 }
 
 type Fish struct {
     Gills string
     Lips string
 }
 
 type Amphibian struct {
     Mammal
     Fish
     Name string
 }
 
 func main() {
     salamander := new(Amphibian)
     salamander.Name = "salamander"
     fmt.Print(salamander.Name + " ")
 
     salamander.Lung = "Lung"
     fmt.Print("has " + salamander.Lung)
 
     salamander.Gills = "Gills"
     fmt.Println(" and " + salamander.Gills)
 
     salamander.Mammal.Lips = "Mammal lips"
     salamander.Fish.Lips = "Fish lips"
     fmt.Println(salamander.Name + " has both " + salamander.Mammal.Lips + " and " + salamander.Fish.Lips)
 }
 
 // salamander has Lung and Gills
 // salamander has both Mammal lips and Fish lips
```
* 不同基类相同 filed 名称的时候可以通过"全路径名"的方式引


### 基于指针实现的扩展
```go
 package main
 
 import (
     "fmt"
 )
 
 type Mammal struct {
     Lung string
 }
 
 type Fish struct {
     Gills string
 }
 
 type Amphibian struct {
     *Mammal
     *Fish
     Name string
 }
 
 func main() {
     salamander := new(Amphibian)
     salamander.Name = "salamander"
     fmt.Print(salamander.Name + " ")
 
     salamander.Lung = "Lung"
     fmt.Print("has " + salamander.Lung)
 
     salamander.Gills = "Gills"
     fmt.Println("and " + salamander.Gills)
 }

 // salamander panic: runtime error: invalid memory address or nil pointer dereference
 // [signal 0xb code=0x1 addr=0x0 pc=0x400f70]
 // 
 // goroutine 1 [running]:
 // runtime.panic(0x499d60, 0x565b28)
 //     /usr/lib/golang/src/pkg/runtime/panic.c:266 +0xb6
 // main.main()
 //     /root/tmp/go/slide/mulity-extend-point.go:26 +0x370
 // exit status 2
```
* 基于指针的扩展需要手动初始化 base struct


### 基于指针实现的扩展
```go
 package main
 
 import (
     "fmt"
 )
 
 type Mammal struct {
     Lung string
 }
 
 type Fish struct {
     Gills string
 }
 
 type Amphibian struct {
     *Mammal
     *Fish
     Name string
 }
 
 func NewAmphibian() *Amphibian {
     ret := new(Amphibian)
     ret.Mammal = new(Mammal)
     ret.Fish = new(Fish)
     return ret
 }
 
 func main() {
     salamander := NewAmphibian()
     salamander.Name = "salamander"
     fmt.Print(salamander.Name + " ")
 
     salamander.Lung = "Lung"
     fmt.Print("has " + salamander.Lung)
 
     salamander.Gills = "Gills"
     fmt.Println(" and " + salamander.Gills)
 }

 // salamander has Lung and Gills
```



## struct field
* struct field 的定义

```go
type StructField struct {
    // Name is the field name.
    // PkgPath is the package path that qualifies a lower case (unexported)
    // field name.  It is empty for upper case (exported) field names.
    // See http://golang.org/ref/spec#Uniqueness_of_identifiers
    Name    string
    PkgPath string

    Type      Type      // field type
    Tag       StructTag // field tag string
    Offset    uintptr   // offset within struct, in bytes
    Index     []int     // index sequence for Type.FieldByIndex
    Anonymous bool      // is an embedded field
}
```


## struct field
* struct field 实例
```go
 package main
 
 import (
     "fmt"
     "reflect"
 )
 
 type T struct {
     A int     `json:key-a`
     B string  `json:key-b`
 }
 
 func main() {
     t := T{23, "skidoo"}
     st := reflect.TypeOf(t)
     for i := 0; i < 2; i++ {
         f := st.Field(i)
         fmt.Printf("index: %d, name: %s, type: %s, pkgPath: %s, Anonymous: %t, tag: %s\n", i,
         f.Name, f.Type, f.PkgPath, f.Anonymous ,f.Tag)
     }
 }
 // index: 0, name: A, type: int,    pkgPath: , Anonymous: false, tag: json:key-a
 // index: 1, name: B, type: string, pkgPath: , Anonymous: false, tag: json:key-b
```



## struct tag
```go
 package main
 
 import (
     "fmt"
     "reflect"
 )
 
 func main() {
     type S struct {
         F string `species:"gopher" color:"blue"`
     }
 
     s := S{}
     st := reflect.TypeOf(s)
     field := st.Field(0)
     fmt.Println(field.Tag.Get("color"), field.Tag.Get("species"))
 
 }

 // blue gopher
```



## 参考文献
* [Golang New & Make](http://no-fucking-idea.com/blog/2014/02/23/golang-new-and-make/)
* [Methods on structs](http://golangtutorials.blogspot.jp/2011/06/methods-on-structs.html)
* [GoLang: Inheritance by embedding](https://geekwentfreak-raviteja.rhcloud.com/blog/2014/03/06/golang-inheritance-by-embedding/)
* [reflect struct](http://golang.org/pkg/reflect/#StructField)
* [package fmt](http://golang.org/pkg/fmt/)
