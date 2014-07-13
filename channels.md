# chan
## 2014-06-27



## 目录

* channel 基本介绍         
* channel 高级用法         
* sync 简介                
* 管道                     
* channel 和锁混用时的隐患 
* 参考文献                 



## chan 基本介绍

```go
 1  ic  := make(chan int)      //a channel that can send and receive an int
 2  sc  := make(chan string)   //a channel hat can send and receive a string
 3  myc := make (chan my_type) //a channel for a custom defined struct type
```

```go
 4  my_channel := make(chan int)
 5
 6  //within some goroutine - to put a value on the channel
 7  my_channel <- 5 
 8
 9  //within some other goroutine - to take a value off the channel
 10 var my_recvd_value int
 11 my_recvd_value = <- my_channel
```

```go
 12 ic_send_only := make (<-chan int) // 此 channel 只能发送 int 数据
 13 ic_recv_only := make (chan<- int) // 此  channel 只能接受 int 数据
```


### 同步和异步 channel
* make(chan int) 这种方式创建的 channe 默认是同步模式的
 * 向 channel 发送数据的 goroutine 会阻塞, 直到别的 goroutine 从 channel 中把数据取走为止
* make(chan int, 100)
 * 异步 channel、可缓冲的 channel。可以同时容纳 100 个 item, 向 channel 中写 101 个 item 的 goroutine 会阻塞
 * 当 channel 中的 item 少于 100 时不会阻塞写 channel 的 goroutine


### 同步 channel 例子
* 制作三个 cake

```go
 package main
 
 import (
     "fmt"
     "time"
     "strconv"
 )
 
 var i int
 
 func makeCakeAndSend(cs chan string) {
     i = i + 1
     cakeName := "Strawberry Cake " + strconv.Itoa(i)
     fmt.Println("Making a cake and sending ...", cakeName)
     cs <- cakeName //send a strawberry cake
 }
 
 func receiveCakeAndPack(cs chan string) {
     s := <-cs //get whatever cake is on the channel
     fmt.Println("Packing received cake: ", s)
 }
 
 func main() {
     cs := make(chan string)
     for i := 0; i<3; i++ {
         go makeCakeAndSend(cs)
         go receiveCakeAndPack(cs)
 
         //sleep for a while so that the program doesn’t exit immediately and output is clear for illustration
         time.Sleep(1 * 1e9)
     }
 }
 //Making a cake and sending ... Strawberry Cake 1 
 //Packing received cake: Strawberry Cake 1 
 //Making a cake and sending ... Strawberry Cake 2 
 //Packing received cake: Strawberry Cake 2 
 //Making a cake and sending ... Strawberry Cake 3 
 //Packing received cake: Strawberry Cake 3

```


### 同步 channel 例子
* 制作三个 cake, goroutine 思维

```go
 package main                                                                                                                                                           
 import (
     "fmt"
     "time"
     "strconv"
 )
 
 func makeCakeAndSend(cs chan string) {
     for i := 1; i<=3; i++ {
         cakeName := "Strawberry Cake " + strconv.Itoa(i)
         fmt.Println("Making a cake and sending ...", cakeName)
         cs <- cakeName //send a strawberry cake
     }   
 }
 
 func receiveCakeAndPack(cs chan string) {
     for {
         s := <-cs //get whatever cake is on the channel
         fmt.Println("Packing received cake: ", s)
     }   
 }
 
 func main() {
     cs := make(chan string)
     go makeCakeAndSend(cs)
     go receiveCakeAndPack(cs)
 
     //sleep for a while so that the program doesn’t exit immediately
     time.Sleep(4 * 1e9)
 }

 //Making a cake and sending ... Strawberry Cake 1 
 //Making a cake and sending ... Strawberry Cake 2 
 //Packing received cake: Strawberry Cake 1 
 //Packing received cake: Strawberry Cake 2 
 //Making a cake and sending ... Strawberry Cake 3 
 //Packing received cake: Strawberry Cake 3
```


## 异步 channel 的例子
```go
 var ...
```
Note: 后面讲管道的时候会讲到, 这里就不再叙述了



## chan 高级用法
### select
```go
 func main() {
     c1 := make(chan string)
     c2 := make(chan string)
     
     go func() {
         for {
             c1 <- "from 1"
             time.Sleep(time.Second * 2)
         }
     }()
     go func() {
         for {
             c2 <- "from 2"
             time.Sleep(time.Second * 3)
         }
     }()
     go func() {
         for {
             select {
             case msg1 := <- c1:
                 fmt.Println(msg1)
             case msg2 := <- c2:
                 fmt.Println(msg2)
             }
         }
     }()
 }
```


### select 一般会附带一个 timeout 分支
```go
 select {
 case msg1 := <- c1:
     fmt.Println("Message 1", msg1)
 case msg2 := <- c2:
     fmt.Println("Message 2", msg2)
 case <- time.After(time.Second):
     fmt.Println("timeout")
 }
```



## sync
* type Mutex
    
* type WaitGroup
    


### Mutex
* func (m *Mutex) Lock()
    
* func (m *Mutex) Unlock()
    

```go
package main
import "sync"
 
var l sync.Mutex
var a string
 
func f() {
    a = "hello, world"
}
 
func main() {
    go f()
    print(a)
}
```


* 共享变量使用前未同步



```go
package main
import "sync"
 
var l sync.Mutex
var a string
 
func f() {
    l.Lock()
    a = "hello, world"
    l.Unlock()
}
 
func main() {
    go f()
    l.Lock()
    print(a)
    l.Unlock()
}
```

* 访问顺序不确定



```go
package main
import "sync"
 
var l sync.Mutex
var a string
 
func f() {
    a = "hello, world"
    l.Unlock()
}
 
func main() {
    l.Lock()
    go f()
    l.Lock()
    print(a)
    l.Unlock()
}
```

* 共享变量使用前进行了同步, 访问顺序确定



### WaitGroup
* func (wg *WaitGroup) Add(delta int)

* func (wg *WaitGroup) Done()

* func (wg *WaitGroup) Wait()


```go
 var wg sync.WaitGroup
 var urls = []string{
         "http://www.golang.org/",
         "http://www.google.com/",
         "http://www.somestupidname.com/",
 }
 for _, url := range urls {
         // Increment the WaitGroup counter.
         wg.Add(1)
         // Launch a goroutine to fetch the URL.
         go func(url string) {
                 // Decrement the counter when the goroutine completes.
                 defer wg.Done()
                 // Fetch the URL.
                 http.Get(url)
         }(url)
 }
 wg.Wait() // Wait for all HTTP fetches to complete.
```




## 管道
### 管道 和 channel 的区别
* 一般的，一个管道就是由一系列的 channel 连接起来的阶段。每个阶段都有执行相应逻辑的 goroutine。在每个阶段中，goroutine

 * 从 channel 读取上游数据
 * 在数据上执行一些操作，通常会产生新的数据
 * 通过 channel 将数据发往下游


### 管道使用模式--生产者/消费者
这个例子的源代码在[这里](http://play.golang.org/p/oKKgOutbQm)
#### 生产者
```go
 func producer(c chan int64, max int) {
     defer close(c)
     for i:= 0; i < max; i ++ {
         c <- time.Now().Unix()
     }
 }
 
```
#### 消费者
```go
 func consumer(c chan int64) {
     var v int64
     ok := true
     for ok {
         if v, ok = <-c; ok {
             fmt.Println(v)
         }
     }
 }
 
```


### 管道使用模式--自增长 ID 生成器
其代码托管在[这里](https://bitbucket.org/mikespook/golib/src/46a4f2a8abcb/autoinc/autoinc.go), [使用示例](https://bitbucket.org/mikespook/golib/src/46a4f2a8abcb/autoinc/autoinc_test.go)
```go
 type AutoInc struct {
     start, step int
     queue chan int
     running bool
 }
  
 func New(start, step int) (ai *AutoInc) {
     ai = &AutoInc{
         start: start,
         step: step,
         running: true,
         queue: make(chan int, 4),
     }
     go ai.process()
     return
 }
  
 func (ai *AutoInc) process() {
     defer func() {recover()}()
     for i := ai.start; ai.running ; i=i+ai.step {
         ai.queue <- i
     }
 }
  
 func (ai *AutoInc) Id() int {
     return <-ai.queue
 }
  
 func (ai *AutoInc) Close() {
     ai.running = false
     close(ai.queue)
 }

```


### 管道使用模式--数字的平方
```go
 func gen(nums ...int) <-chan int {
     out := make(chan int)
     go func() {
         for _, n := range nums {
             out <- n
         }
         close(out)
     }()
     return out
 }

 // out := make(chan int, len(nums))

 func sq(in <-chan int) <-chan int {
     out := make(chan int)
     go func() {
         for n := range in {
             out <- n * n
         }
         close(out)
     }()
     return out
 }

 func main() {
     // Set up the pipeline.
     c := gen(2, 3)
     out := sq(c)
  
     // Consume the output.
     fmt.Println(<-out) // 4
     fmt.Println(<-out) // 9
 }
```

Note: 第一阶段，gen 函数，是一个将数字列表转换到一个 channel 中的函数。使用一个 goroutine 来执行 gen，将数字发送到 channel，并在所有数字都发送完后关闭 channel。
第二个阶段，sq，从上面的channel接收数字，并返回一个包含所有收到数字的平方的channel。在上游channel关闭后，这个阶段已经往下游发送完所有的结果，然后关闭输出channel：
 main函数建立这个管道，并执行第一个阶段，从第二个阶段接收结果并逐个打印，直到channel被关闭。


### 管道使用模式--扇出扇入
* 多个函数(一般是多个 goroutine)可以从同一个 channel 读取数据，直到这个 channel 关闭，这叫扇出。
Note: 这是一种多个工作实例分布式地协作以并行利用 CPU 和 I/O 的方式。
* 一个函数可以从多个输入读取并处理数据，直到所有的输入channel都被关闭。这个函数会将所有输入channel导入一个单一的channel。这个单一的channel在所有输入channel都关闭后才会关闭。这叫做扇入。


### 管道使用模式--扇出扇入
```go
 func merge(cs ...<-chan int) <-chan int {
     var wg sync.WaitGroup
     out := make(chan int)
  
     // Start an output goroutine for each input channel in cs.  output
     // copies values from c to out until c is closed, then calls wg.Done.
     output := func(c <-chan int) {
         for n := range c {
             out <- n
         }
         wg.Done()
     }
     wg.Add(len(cs))
     for _, c := range cs {
         go output(c)
     }
  
     // Start a goroutine to close out once all the output goroutines are
     // done.  This must start after the wg.Add call.
     go func() {
         wg.Wait()
         close(out)
     }()
     return out
 }
```


### 停止的艺术
* 我们所有的管道函数都遵循一种模式：

 * 发送者在发送完毕时关闭其输出 channel。
    
 * 接收者持续从输入管道接收数据直到输入管道关闭。
    
* 这种模式使得每一个接收函数都能写成一个 range 循环，保证所有的 goroutine 在数据成功发送到下游后就关闭。
    


### 停止的艺术
```go
 func main() {
     ......
     // Consume the first value from output.
     out := merge(c1, c2)
     fmt.Println(<-out) // 4 or 9
     return
     // Since we didn't receive the second value from out,
     // one of the output goroutines is hung attempting to send it.
 }
```
Note: 
 * 但是在真实的案例中，并不是所有的输入数据都需要被接收处理。有些时候是故意这么设计的：接收者可能只需要数据的子集就够了；或者更一般的，因为输入数据有错误而导致接收函数提早退出。上面任何一种情况下，接收者都不应该继续等待后续的数据到来，并且我们希望上游函数停止生成后续步骤已经不需要的数据。

 * 在我们的管道例子中，如果一个阶段无法消费所有的输入数据，那些发送这些数据的goroutine就会一直阻塞下去：


### 显示取消
```go
 func main() {
     in := gen(2, 3)
  
     // Distribute the sq work across two goroutines that both read from in.
     c1 := sq(in)
     c2 := sq(in)
  
     // Consume the first value from output.
     done := make(chan struct{}, 2)
     out := merge(done, c1, c2)
     fmt.Println(<-out) // 4 or 9
  
     // Tell the remaining senders we're leaving.
     done <- struct{}{}
     done <- struct{}{}
 }
```
Note:
  传递 done channel 到上游 goroutine


### 显示取消
```go
 func merge(done <-chan struct{}, cs ...<-chan int) <-chan int {
     var wg sync.WaitGroup
     out := make(chan int)
  
     // Start an output goroutine for each input channel in cs.  output
     // copies values from c to out until c or done is closed, then calls
     // wg.Done.
     output := func(c <-chan int) {
         defer wg.Done()
         for n := range c {
             select {
             case out <- n:
             case <-done:
                 return
             }
         }
     }
 }
```
Note:
 收到 done 时主动退出


### channe 构建的建议
* 管道构建的指导思想如下：

 * 每一个阶段在所有发送操作完成后关闭输出channel。
    
 * 每一个阶段持续从输入channel接收数据直到输入channel被关闭或者生产者被解除阻塞（译者：生产者退出）。
    

* 管道解除生产者阻塞有两种方法： 
    * 保证有足够的缓存空间存储将要被生产的数据
    
    * 显式的通知生产者消费者要取消接收数据
    



## chan 和锁混用时的隐患

```go
 package main
 
 import (
     "fmt"
 )
 
 var routineNum int = 0
 
 func main() {
     ok := make(chan bool)
     runTask(ok)
     <-ok
     fmt.Println("go program end")
 }
 
 func runTask(ok chan bool) error{
     var i int
     for i=0; i<1000; i++ {
         routineNum = routineNum + 1
         fmt.Println("routineNum %d ", routineNum)
         go exec(ok)
     }
     return nil
 }
 
 func exec(ok chan bool) {
     routineNum = routineNum - 1
     if routineNum <=0 {
         ok<-true
     }
 }
 
// ...
 //routineNum %d  102
 //routineNum %d  103
 //routineNum %d  104
 //routineNum %d  105
 //routineNum %d  106
 //routineNum %d  107
 //routineNum %d  108
 //routineNum %d  109
 //routineNum %d  110
 //routineNum %d  111
 //routineNum %d  112
```


* 为什么创建的 goroutine 不到 1000?



## chan 和锁混用时的隐患

* 使用锁


```go
 
 package main
 
 import (
     "fmt"
     "sync"
 //    "time"
 )
 
 var routineNum int = 0
 var lc sync.Mutex
 
 func main() {
     ok := make(chan bool)
     runTask(ok)
     <-ok
     fmt.Println("go program end")
 }
 
 func runTask(ok chan bool) error{
     var i int
     for i=0; i<1000; i++ {
         lc.Lock()
         routineNum = routineNum + 1
         lc.Unlock()
         fmt.Printf("routineNum %d \n", routineNum)
         go exec(ok)
     }
     return nil
 }
 
 func exec(ok chan bool) {
     lc.Lock()
     routineNum = routineNum - 1
     if routineNum <=0 {
         ok<-true
     }
     lc.Unlock()
 }
 
 //routineNum 59
 //routineNum 60
 //routineNum 61
 //routineNum 62
 //routineNum 63
 //routineNum 64
 //routineNum 65
 //routineNum 66
```



## chan 和锁混用时的隐患
### 死锁

```go
 routineNum 38 
 routineNum 39 
 routineNum 40 
 routineNum 41 
 fatal error: all goroutines are asleep - deadlock!
 
 goroutine 1 [semacquire]:
 sync.runtime_Semacquire(0x8174e0c)
     /usr/local/go/src/pkg/runtime/sema.goc:199 +0x34
 sync.(*Mutex).Lock(0x8174e08)
     /usr/local/go/src/pkg/sync/mutex.go:66 +0xce
 main.runTask(0x1841b060, 0x0, 0x0)
     /root/tmp/go/chan-lock-2.go:22 +0x3e
 main.main()
     /root/tmp/go/chan-lock-2.go:14 +0x48
 
 goroutine 43 [chan send]:
 main.exec(0x1841b060)
     /root/tmp/go/chan-lock-2.go:35 +0x58
 created by main.runTask
     /root/tmp/go/chan-lock-2.go:26 +0xd8
 exit status 2
```


## chan 和锁混用时的隐患
### 不在 "锁中" 嵌套 "chan 锁"

```go
 func exec(ok chan bool) {
     lc.Lock()
     routineNum = routineNum - 1
     if routineNum <=0 {
         lc.Unlock()
         ok<-true
     }else{
         lc.Unlock()
     }
 }
```

* 竞太条件不能嵌套使用




## 参考文献
* [effective go channels](http://golang.org/doc/effective_go.html#channels)
* [channels in go](http://golangtutorials.blogspot.jp/2011/06/channels-in-go.html)
* [Introduction To Go Channels Again](http://openmymind.net/Introduction-To-Go-Channels-Again/)
* [Concurrency](http://www.golang-book.com/10/index.htm)
* [GOLANG FUNNY: PLAY WITH CHANNEL](http://mikespook.com/2012/05/golang-funny-play-with-channel/)
* [Go Concurrency Patterns: Pipelines and cancellation](http://blog.golang.org/pipelines)
* [Go 并发模式：管道和显式取消](http://blog.jobbole.com/65552/)
* [Broadcasting values in Go with linked channels](http://rogpeppe.wordpress.com/2009/12/01/concurrent-idioms-1-broadcasting-values-in-go-with-linked-channels/)
* [Package sync](http://golang.org/pkg/sync/#example_WaitGroup)
* [how to stop a goroutine](http://stackoverflow.com/questions/6807590/how-to-stop-a-goroutine)
* [Curious Channels](http://dave.cheney.net/2013/04/30/curious-channels)
* [绝妙的 channel](http://mikespook.com/2013/05/%E7%BF%BB%E8%AF%91%E7%BB%9D%E5%A6%99%E7%9A%84-channel/)
