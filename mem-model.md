# go 内存模型
## 冬岛
## 2014-06-24



## 目录
* 简单说明          <!-- .element: class="fragment" data-fragment-index="1" -->
* 线程和 goroutine 比较         <!-- .element: class="fragment" data-fragment-index="2" -->
* 之前发生          <!-- .element: class="fragment" data-fragment-index="3" -->
* 错误的同步        <!-- .element: class="fragment" data-fragment-index="5" -->
* 参考文档          <!-- .element: class="fragment" data-fragment-index="6" -->



## 简单说明
这篇 Go 的内存模型是来阐述，在一个 goroutine 中写入的一个变量的值在何种情况下会被另外一个 goroutine 察觉到



## 之前发生
为了说明读写，我们定义一个之前发生的偏序关系。

1. 如果事件 e1 在 e2 之前发生，那么我们也说，e2 在 e1 的之后发生。
    <!-- .element: class="fragment" data-fragment-index="1" -->
1. e1 先于 e2 发生，而 e1 的结束是否先于 e2 的开始，这不确定
    <!-- .element: class="fragment" data-fragment-index="2" -->
1. 如果 e1 既不是在 e2 的之前发生，也不是在 e2 的之后发生，我们就说，e1 和 e2 是并发的
    <!-- .element: class="fragment" data-fragment-index="3" -->


只要满足下面两个条件，某一个读 r 访问到另外的一个写 w 的结果都是允许的, 允许的意思是可以访问到，但也可以访问不到

* 读 r 不在 w 的之前发生；
    <!-- .element: class="fragment" data-fragment-index="1" -->
* 没有其他的写 w' 位于在 w 的之后，r 之前
    <!-- .element: class="fragment" data-fragment-index="2" -->


在单个的 goroutine 中，之前发生的这个顺序关系就是程序所描述的顺序


为了保证一个读 r 的结果正好就是某一个写 w ，需要保证 r 能看到的 w 只有一个；也就是说要满足下面两个条件：

* w 发生在 r 之前；
    <!-- .element: class="fragment" data-fragment-index="1" -->
* 其他的写 w' 要么位于 w 之前，要么位于 r 之后
    <!-- .element: class="fragment" data-fragment-index="2" -->


### 单个 goroutine
* -- w0 ---- r1 -- w1 ---- w2 -----  r2 ---- r3 ------>


### 2 个 goroutine
* -- w0 ---- r1 ---- r2 ---- w3 -----  w4 ------ r5 -->
* -- w1 ------- w2 ---- r3 -----  r4 ------ w5 ------->

* 单个 routine 都是有先后关系的，但多个 routine 之间: r1 和 w2 的先后顺序是神马？即使从开始时间看上去 w2 > r1，但在 go 中 w2 既不是先于 r1 也不是后于 r1 的, 准确的说, w2 和 r1 是并发的
    <!-- .element: class="fragment" data-fragment-index="2" -->
* 并发的 r/w 值是不确定的。比如，r3 读的结果是多少呢？ <!-- .element: class="fragment" data-fragment-index="3" -->
 * 可能是前面的 w2 的值，也可能是上面的 w3 的值，或是 w4 的值； <!-- .element: class="fragment" data-fragment-index="4" -->
 * 而 r5 的值，可能是 w4 的值，也能是 w1、w2、w5 的值 <!-- .element: class="fragment" data-fragment-index="5" -->



### goroutine 交叉
* -- r0 ---- r1 ----|----- r2 ----------|---w5---r5-> 
* -- w1 ----- w2 ---|-- r3 ---r4 -- w4 -|-->

* 现在上面添加了两个交点,这样的话，r3 就是后于 r1 的，先于 w5 的；
    <!-- .element: class="fragment" data-fragment-index="2" -->
* r2 前面的写入的是 w2，而并发的有 w4，所以 r2 的值是不确定的，可以是 w2，也可以是 w4 ！！而 r4 前面写入的是 w2，与它并发的没有写入，所以 r4 读的值是 w2 ！！
    <!-- .element: class="fragment" data-fragment-index="3" -->


### 结论
如果不加同步控制的话，所有的 routine 都是并发的，这样的话，main 函数以外的这些并发的 go routine 跟 main 函数没有任何关系！只有加上同步控制，比如锁、比如管道，这样的话，routine 之间便打上了“结点”，它们之间便有了先于/后于的顺序，但是在两个“结点”之间的部分，同样还是没有先后关系的。



## 错误的同步


### 线程的同步--变量操作 
* 写一个内存变量比如 a = a+1; 该操作会拆分成多步 <!-- .element: class="fragment" data-fragment-index="3" -->
 * 从内存 load 到寄存器    <!-- .element: class="fragment" data-fragment-index="4" -->
 * 在寄存器中 +1           <!-- .element: class="fragment" data-fragment-index="5" -->
 * 从寄存器 store 到内存   <!-- .element: class="fragment" data-fragment-index="6" -->


#### 多线程同步
* 初始值: a = 10
* 线程 1 写: a = a + 1 <!-- .element: class="fragment" data-fragment-index="1" -->
* 线程 2 写: a = a + 1 <!-- .element: class="fragment" data-fragment-index="2" -->
* 线程 3 读: a = ?     <!-- .element: class="fragment" data-fragment-index="3" -->


### go routine 的特点-1
* goroutine 之间虽然数据是共享的, 但如果不加同步机制的话, 这种共享在 goroutine 之间可能是不可见

```go
var a int = 10

func f() {
    a = a+1
}

func g() {
    a = a+1
}

func main() {
    go f()
    go g()
    time.sleep(3)
    fmt.Println("a values: %d", a)
}
```
<!-- .element: class="fragment" data-fragment-index="2" -->


### go routine 的特点-2
编译器或是处理器是可以重新排列变量的读写顺序，只要排列后的顺序并没有违反语言规范的定义就可以

* a = 1; b = 2; 因为可能会重新排序，所以一个 goroutine 中看到的顺序和另外一个 goroutine 看到的可能是不一样的
    <!-- .element: class="fragment" data-fragment-index="2" -->

```go
var a string
var done bool
func setup() {
        a = "hello, world"
        done = true
}
func main() {
        go setup()
        for !done {
        }
        fmt.Println(a)
}
```
<!-- .element: class="fragment" data-fragment-index="3" -->


```go
type T struct {
        msg string
}
 
var g *T
 
func setup() {
        t := new(T)
        t.msg = "hello, world"
        g = t
}
 
func main() {
        go setup()
        for g == nil {
        }
        print(g.msg)
}

```



## 参考文档
* [Go 的内存模型](http://ilovers.sinaapp.com/article/go%E7%9A%84%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B)
* [The Go Memory Model](http://golang.org/ref/mem)



# 谢谢!
