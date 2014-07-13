# 数据类型
## 冬岛
## 2014-06-25



## 目录

* 数组              
* 值和指针          
* slice 结构        
* go 数据类型举例   
* 参考文献          




## 数组

* 数组有两个关键属性:

 * 数据类型  
 * 长度      
* 数组的长度是固定的, 一旦设定就不能修改, 长度也数据类型的一部分 [4]int 和 [5]int 是两个不同的数组, 因为他们的长度不同


![数组示例](http://218.244.157.157/images/go-slices-usage-and-internals_slice-array.png "数组示例")


* 数组是一个 values 类型, 和 C 不一样 go 的数组变量代表整个数组, 而不是一个指向第一个元素的指针



### 举例
```go
package main

import "fmt"

func main(){
    var a [4]string = [4]string{"string-a","string-b","string-c","string-d"}
    var b [4]string
    b = a
    fmt.Printf("修改前 array a: %s, array b: %s\n", a, b)
    b[2] = "my-self-string"
    fmt.Printf("修改后 array a: %s, array b: %s\n", a, b)
}

// 输出
// 修改前 array a: [string-a string-b string-c string-d], array b: [string-a string-b string-c string-d]
// 修改后 array a: [string-a string-b string-c string-d], array b: [string-a string-b my-self-string string-d]
```



## 值和指针

values(值)类型的变量是可以通过 *赋值* 进行 *复制* 的, 比如:

* int values 类型的变量
```go
var a int = 10
var b int = a
a = 12
fmt.Printf("a: %d b: %d\n", a, b )
// 输出: a: 12 b: 10
// 因为 int 是 values 类型的变量, 不是指针类型 所以 a 的修改不影响 b
``` 

* 同样: array、string、bool、int64 等 int 系列都是 values 类型的变量
    



## go 数据类型举例


### slice 结构

* slice 是对 array 的进一步抽象, 其使用方法和 array 不同, slice 有三个关键属性: 
 * 一个指向数组的指针
    
 * 指向的 array 的长度
    
 * 最大容量
    
 * slice 的结构如下:
    

![slice-struct](http://218.244.157.157/images/go-slices-usage-and-internals_slice-struct.png "slice struct")
    


### slice 结构
* s := make([]byte, 5), s 的结构如下
![slice-struct](http://218.244.157.157/images/go-slices-usage-and-internals_slice-1.png "数组示例")
* 使用 make 创建 slice make(type, length [,capacity ])
 * length slice 的初始长度
 * capacity slice 的最大长度
 * 如果 capacity 未指定, 默认和 length 一样
* 在 capacity 内 slice 是可以通过下标 读/写, 超过 capacity 的部分可以通过 append 操作


### slice 结构

* s = s[2:4] s 指向数据新的位置
* slice 操作的是指向数组的指针

![slice-struct](http://218.244.157.157/images/go-slices-usage-and-internals_slice-2.png "数组示例")

```go
d := [...]byte{'r', 'o', 'a', 'd','e'}
e := d[2:4] 
// e == []byte{'a', 'd'}
e[1] = 'm'
// e == []byte{'a', 'm'}
// d == []byte{'r', 'o', 'a', 'm','e'}
```


### slice 结构
```go
d := [...]byte{'r', 'o', 'a', 'd', 'e'}
s := make(byte,3)
s = d[:cap(s)]
```
![slice-struct](http://218.244.157.157/images/go-slices-usage-and-internals_slice-3.png "数组示例")
 

* cap() 获取切片的容量
 
* len() 获取切片的长度
 


### slice len

```go
 package main
 
 import "fmt"
 
 func main() {
    var s []int = make([]int, 3, 10)
    s[0] = 1
    fmt.Printf("s len %d, s value %s\n", len(s), s)
    //s len 3, s value [%!s(int=1) %!s(int=0) %!s(int=0)]

    s = append(s, 2)
    fmt.Printf("s len %d, s value %s\n", len(s), s)
    //s len 4, s value [%!s(int=1) %!s(int=0) %!s(int=0) %!s(int=2)]

    var s1 []int
    s1 = append(s1, 2)
    fmt.Printf("s1 len %d, s1 value %s\n", len(s1), s1)
    //s1 len 1, s1 value [%!s(int=2)]

    s1 = append(s1, 3)
    fmt.Printf("s1 len %d, s1 value %s\n", len(s1), s1)
    //s1 len 2, s1 value [%!s(int=2) %!s(int=3)]

    var s3 []int = make([]int, 0, 10)
    s3 = append(s3, 2)
    fmt.Printf("s3 len %d, s3 value %s\n", len(s3), s3)
    //s3 len 1, s3 value [%!s(int=2)]

    s3[1] = 1
    fmt.Printf("s3 len %d, s3 value %s\n", len(s3), s3)
    //panic: runtime error: index out of range
    //
    //goroutine 1 [running]:
    //runtime.panic(0x49aa00, 0x566d17)
    //    /usr/lib/golang/src/pkg/runtime/panic.c:266 +0xb6
    //main.main()
    //    /root/tmp/go/slice-test.go:23 +0x8a4
    //exit status 2
 }

```


### map
```go
 package main
 
 import "fmt"
 
 func main() {
     m := make( map[string]string)
     m["str-1"] = "string-m-1"
     m["str-2"] = "string-m-2"
     m["str-3"] = "string-m-3"
 
     x := m
     m["str-1"] = "string-x-1"
     m["str-2"] = "string-x-2"
     m["str-3"] = "string-x-3"
     fmt.Println("The m value: ", m)
     fmt.Println("The x value ", x)
     // The m value:  map[str-1:string-x-1 str-2:string-x-2 str-3:string-x-3]
     // The x value  map[str-1:string-x-1 str-2:string-x-2 str-3:string-x-3]
 }

```


### struct

```go
  package main
  
  import "fmt"
  
  type T struct {
      Id   int
      Name string
  }
  
  func main() {
     var a = T{1, "one"}
     var b = T{2, "two"}
  
     fmt.Println(a, b)
     //{1 one} {2 two}
     a = b
     fmt.Println(a, b)
     //{2 two} {2 two}
 
     var c = &T{1, "one"}
     var d = &T{2, "two"}
 
     fmt.Println(c, d)
     //&{1 one} &{2 two}
     *c = *d
     fmt.Println(c, d)
     //&{2 two} &{2 two}
  }
 
```



## 参考文献
* [Go Slices: usage and internals](http://blog.golang.org/go-slices-usage-and-internals)
