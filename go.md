# go:

* [深拷贝和浅拷贝](#深拷贝和浅拷贝)
* [Int](#Int)
	* [go里面的int和int32是同一个概念吗](#go里面的int和int32是同一个概念吗)
	* [uint型变量值分别为 1，2，它们相减的结果是多少](#uint型变量值分别为 1，2，它们相减的结果是多少)
* [string](#string)
	* [如何高效地拼接字符串](#如何高效地拼接字符串)
* [Slice](#Slice)
	* [Slice结构体](#Slice结构体)
	* [Go的Slice如何扩容](#Go的Slice如何扩容)
	* [如何判断 2 个字符串切片（slice) 是相等的](#如何判断 2 个字符串切片（slice) 是相等的)
	* [使用array还是slice？](#使用array还是slice？)
* [Map](#Map)
	* [map的底层实现](#map的底层实现。)
	* [如何判断 map 中是否包含某个 key](#如何判断 map 中是否包含某个 key)
* [Chan](#Chan)
	* [CHAN结构体](#CHAN结构体)
	* [读写流程](#读写流程)
	* [无缓冲Chan的发送和接收是否同步?](#无缓冲Chan的发送和接收是否同步?)
	* [channel 为什么它可以做到线程安全？](#channel 为什么它可以做到线程安全？)
	* [channel 死锁的场景 ](#channel 死锁的场景)
	* [Channel是同步的还是异步的？](#Channel是同步的还是异步的？)
	* [读写channel应该先关哪个？ ](#读写channel应该先关哪个？)
* [Tag](#Tag)
	* [tag 的用处](#tag 的用处)
	* [Go解析Tag是怎么实现的](#Go解析Tag是怎么实现的)
	* [如何获取一个结构体的所有tag](#如何获取一个结构体的所有tag)
* [错误处理](#错误处理)
	* [error](#error)
	* [何时会发生Panic](#何时会发生Panic)
	* [Panic会执行什么](#Panic会执行什么)
	* [defer可以捕获goroutine的子goroutine吗](#defer可以捕获goroutine的子goroutine吗)

### 深拷贝和浅拷贝

- [ ] 深拷贝：拷贝的是值，开辟了新的内存空前，修改操作不影响原先的内存
- [ ] 浅拷贝：拷贝的是指针，指向的还是原来的内存空间，修改操作直接作用在原内存空间上

### Int

##### **go里面的int和int32是同一个概念吗**

> go语言中的`int的大小是和操作系统位数相关的`，如果是32位操作系统，int类型的大小就是4字节。如果是64位操作系统，int类型的大小就是8个字节

##### uint型变量值分别为 1，2，它们相减的结果是多少

> 结果会`溢出`，如果是32位系统，结果是==2^32-1^==，如果是64位系统，结果==2^64-1^==

### string

##### 如何高效地拼接字符串

- [ ] "+"
- [ ] fmt.Sprintf
- [ ] strings.Builder
- [ ] bytes.Buffer
- [ ] strings.Join

> strings.Join ≈ strings.Builder > bytes.Buffer >  "+" > fmt.Sprintf

### Slice

##### Slice结构体

```go
type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```

#####  Go的Slice如何扩容

**slice 实现原理**

在使用 append 向 slice 追加元素时，若 slice 空间不足则会发生扩容，扩容会重新分配一块更大的内存，将原 slice 拷贝到新 slice ，然后返回新 slice。扩容后再将数据追加进去。

扩容操作只对容量，扩容后的 slice 长度不变，容量变化规则如下：

- `若 slice 容量小于1024个元素，那么扩容的时候slice的cap就翻番`，乘以2；`一旦元素个数超过1024个元素，增长因子就变成1.25，即每次增加原来容量的四分之一`。
- `若 slice 容量够用，则将新元素追加进去，slice.len++，返回原 slice`
- `若 slice 容量不够用，将 slice 先扩容，扩容得到新 slice，将新元素追加进新 slice，slice.len++，返回新 slice。`

##### 如何判断 2 个字符串切片（slice) 是相等的

```go
reflect.DeepEqual()
```



##### 使用array还是slice？

>  在Go语言中只存在值传递，要给函数传递一个有100w个元素的array时，直接使用array传递的效率是非常低的，因为array是值拷贝。这时就应该使用slice作为参数，就相当于传递了一个指针。
>
>  如果元素数量比较少，使用array还是slice作为参数，效率差别并不大。

- [ ] 元素较多时使用Slice

### Map

##### map的底层实现。

源码位于`src\runtime\map.go` 中。

go的map和C++map不一样，底层实现是`哈希表`，包括两个部分：**hmap**和**bucket**。

hmap结构体如图：

```text
type hmap struct {
    count     int //map元素的个数，调用len()直接返回此值
    
    // map标记:
    // 1. key和value是否包指针
    // 2. 是否正在扩容
    // 3. 是否是同样大小的扩容
    // 4. 是否正在 `range`方式访问当前的buckets
    // 5. 是否有 `range`方式访问旧的bucket
    flags     uint8 
    
    B         uint8  // buckets 的对数 log_2, buckets 数组的长度就是 2^B
    noverflow uint16 // overflow 的 bucket 近似数
    hash0     uint32 // hash种子 计算 key 的哈希的时候会传入哈希函数
    buckets   unsafe.Pointer // 指向 buckets 数组，大小为 2^B 如果元素个数为0，就为 nil
    
    // 扩容的时候，buckets 长度会是 oldbuckets 的两倍
    oldbuckets unsafe.Pointer // bucket slice指针，仅当在扩容的时候不为nil
    
    nevacuate  uintptr // 扩容时已经移到新的map中的bucket数量
    extra *mapextra // optional fields
}
```

里面最重要的是buckets（桶）。buckets是一个指针，最终它指向的是一个结构体：

```text
// A bucket for a Go map.
type bmap struct {
    tophash [bucketCnt]uint8
}
```

每个bucket固定包含8个key和value(可以查看源码bucketCnt=8).实现上面是一个固定的大小连续内存块，分成四部分：每个条目的状态，8个key值，8个value值，指向下个bucket的指针。

创建哈希表使用的是`makemap`函数.map 的一个关键点在于，**哈希函数**的选择。在程序启动时，会检测 cpu 是否支持 aes，如果支持，则使用 aes hash，否则使用 memhash。这是在函数 alginit() 中完成，位于路径：`src/runtime/alg.go` 下。

map查找就是将key哈希后得到64位（64位机）用最后B个比特位计算在哪个桶。在 bucket 中，从前往后找到第一个空位。这样，在查找某个 key 时，先找到对应的桶，再去遍历 bucket 中的 key。

关于map的查找和扩容可以参考[map的用法到map底层实现分析](https://link.zhihu.com/?target=https%3A//blog.csdn.net/chenxun_2010/article/details/103768011%3Futm_medium%3Ddistribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0.pc_relevant_aa%26spm%3D1001.2101.3001.4242.1%26utm_relevant_index%3D3)。

##### 如何判断 map 中是否包含某个 key

```go
var sample map[int]int
if _, ok := sample[10];ok{

}else{

}
```



### Chan

##### CHAN结构体

```go
type hchan struct {
 qcount   uint  // 队列中的总元素个数
 dataqsiz uint  // 环形队列大小，即可存放元素的个数
 buf      unsafe.Pointer // 环形队列指针
 elemsize uint16  //每个元素的大小
 closed   uint32  //标识关闭状态
 elemtype *_type // 元素类型
 sendx    uint   // 发送索引，元素写入时存放到队列中的位置

 recvx    uint   // 接收索引，元素从队列的该位置读出
 recvq    waitq  // 等待读消息的goroutine队列
 sendq    waitq  // 等待写消息的goroutine队列
 lock mutex  //互斥锁，chan不允许并发读写
}
```

channel内部是一个循环链表。内部包含buf, sendx, recvx, lock ,recvq, sendq几个部分；

buf是有缓冲的channel所特有的结构，用来存储缓存数据。是个循环链表；

- sendx和recvx用于记录buf这个循环链表中的发送或者接收的index；
- lock是个互斥锁；
- recvq和sendq分别是接收(<-channel)或者发送(channel <- xxx)的goroutine抽象出来的结构体(sudog)的队列。是个双向链表。

channel是**线程安全**的。

##### 读写流程

**向 channel 写数据:**

若等待接收队列 recvq 不为空，则缓冲区中无数据或无缓冲区，将直接从 recvq 取出 G ，并把数据写入，最后把该 G 唤醒，结束发送过程。

若缓冲区中有空余位置，则将数据写入缓冲区，结束发送过程。

若缓冲区中没有空余位置，则将发送数据写入 G，将当前 G 加入 sendq ，进入睡眠，等待被读 goroutine 唤醒。

**从 channel 读数据**

若等待发送队列 sendq 不为空，且没有缓冲区，直接从 sendq 中取出 G ，把 G 中数据读出，最后把 G 唤醒，结束读取过程。

如果等待发送队列 sendq 不为空，说明缓冲区已满，从缓冲区中首部读出数据，把 G 中数据写入缓冲区尾部，把 G 唤醒，结束读取过程。

如果缓冲区中有数据，则从缓冲区取出数据，结束读取过程。

将当前 goroutine 加入 recvq ，进入睡眠，等待被写 goroutine 唤醒。

**关闭 channel**

1.关闭 channel 时会将 recvq 中的 G 全部唤醒，本该写入 G 的数据位置为 nil。将 sendq 中的 G 全部唤醒，但是这些 G 会 panic。

panic 出现的场景还有：

- 关闭值为 nil 的 channel
- 关闭已经关闭的 channel
- 向已经关闭的 channel 中写数据

##### 无缓冲 Chan 的发送和接收是否同步?

```go
// 无缓冲的channel由于没有缓冲发送和接收需要同步
ch := make(chan int)   
//有缓冲channel不要求发送和接收操作同步
ch := make(chan int, 2)  
```

channel 无缓冲时，发送阻塞直到数据被接收，接收阻塞直到读到数据；channel有缓冲时，当缓冲满时发送阻塞，当缓冲空时接收阻塞。



##### channel 为什么它可以做到线程安全？

- [ ] Channel  可以理解是一个先进先出的队列，通过管道进行通信,发送一个数据到Channel和从Channel接收一个数据都是原子性的。不要通过共享内存来通信，而是通过通信来共享内存，前者就是传统的加锁，后者就是Channel。设计Channel的主要目的就是在多任务间传递数据的，本身就是安全的。

##### channel 死锁的场景 

- 当一个`channel`中没有数据，而直接读取时，会发生死锁：

```text
q := make(chan int,2)
<-q
```

解决方案是采用select语句，再default放默认处理方式：

```text
q := make(chan int,2)
select{
   case val:=<-q:
   default:
         ...

}
```

- 当channel数据满了，再尝试写数据会造成死锁：

```text
q := make(chan int,2)
q<-1
q<-2
q<-3
```

解决方法，采用select

```text
func main() {
	q := make(chan int, 2)
	q <- 1
	q <- 2
	select {
	case q <- 3:
		fmt.Println("ok")
	default:
		fmt.Println("wrong")
	}

}
```

- 向一个关闭的channel写数据。

参考资料：[Golang关于channel死锁情况的汇总以及解决方案](https://link.zhihu.com/?target=https%3A//blog.csdn.net/qq_35976351/article/details/81984117)

##### Channel是同步的还是异步的？

Channel是`异步`进行的, channel存在`3种状态`：

- `nil`，未初始化的状态，只进行了声明，或者手动赋值为nil
- `active`，正常的channel，可读或者可写
- `closed`，已关闭，千万不要误认为关闭channel后，channel的值是nil

| 操作     | 一个零值nil通道 | 一个非零值但已关闭的通道 | 一个非零值且尚未关闭的通道 |
| -------- | --------------- | ------------------------ | -------------------------- |
| 关闭     | 产生panic       | 产生panic                | 成功关闭                   |
| 发送数据 | 永久阻塞        | 产生panic                | 阻塞或者成功发送           |
| 接收数据 | 永久阻塞        | 永不阻塞                 | 阻塞或者成功接收           |

##### 读写channel应该先关哪个？ 

应该写channel先关。因为对于已经关闭的channel只能读，不能写。



### Tag

##### tag 的用处

 tag可以为结构体成员提供属性。常见的：

1. json序列化或反序列化时字段的名称
2. db: sqlx模块中对应的数据库字段名
3. form: gin框架中对应的前端的数据字段名
4. binding: 搭配 form 使用, 默认如果没查找到结构体中的某个字段则不报错值为空, binding为 required 代表没找到返回错误给前端

##### Go解析Tag是怎么实现的

Go解析tag采用的是**反射**。

具体来说使用reflect.ValueOf方法获取其反射值，然后获取其Type属性，之后再通过Field(i)获取第i+1个field，再.Tag获得Tag。

反射实现的原理在: `src/reflect/type.go`中

##### 如何获取一个结构体的所有tag

利用反射：

```go
type Author struct {
	Name         int      `json:Name`
	Publications []string `json:Publication,omitempty`
}

func main() {
	t := reflect.TypeOf(Author{})
	for i := 0; i < t.NumField(); i++ {
		name := t.Field(i).Name
		s, _ := t.FieldByName(name)
		fmt.Println(s.Tag)
	}
}

```

### 

### 错误处理

##### error

`error`是一个接口，必须实现`Error()`方法

##### 何时会发生Panic

- [ ] 运行错误时会导致panic，比如数组越界，除0
- [ ] 程序主动调用panic()

##### Panic会执行什么

- [ ] 逆序执行当前goroutine的defer链(recover从这里接入)
- [ ] 打印错误信息和调用堆栈
- [ ] 调用exit(2)结束整个进程

```go
func soo(a, b int) {
    defer func() {
        //recover必须在defer中才能生效
        if err := recover(); err != nil {
            fmt.Printf("soo函数中发生了panic：%s\n", err)
        }
    }()
    panic(errors.New("my error"))
}
```

##### defer可以捕获goroutine的子goroutine吗

`不可以`。它们处于不同的调度器P中。对于子goroutine，正确的做法是：

1. 必须通过 defer 关键字来调用 recover()。
2. 当通过 goroutine 调用某个方法，一定要确保内部有 recover() 机制。

##### 如果若干个goroutine，有一个panic会怎么做？

> 有一个panic，那么剩余goroutine也会退出。

### CSP模型

>  CSP 模型是**以通信的方式来共享内存**，不同于传统的多线程通过共享内存来通信。用于描述两个独立的并发实体通过共享的通讯 channel (管道)进行通信的并发模型。

### context 结构原理

##### 用途

- [x] Context（上下文）是Golang应用开发常用的`并发控制技术` ，它可以控制一组呈树状结构的goroutine，每个goroutine拥有相同的上下文。`Context 是并发安全的`，主要是用于控制多个协程之间的协作、取消操作。

##### 结构体

```go
type Context interface {
   Deadline() (deadline time.Time, ok bool)
   Done() <-chan struct{}
   Err() error
   Value(key interface{}) interface{}
}
```

- 「Deadline」 方法：可以获取设置的截止时间，返回值 deadline 是截止时间，到了这个时间，Context 会自动发起取消请求，返回值 ok 表示是否设置了截止时间。
- 「Done」 方法：返回一个只读的 channel ，类型为 struct{}。如果这个 chan 可以读取，说明已经发出了取消信号，可以做清理操作，然后退出协程，释放资源。
- 「Err」 方法：返回Context 被取消的原因。
- 「Value」 方法：获取 Context 上绑定的值，是一个键值对，通过 key 来获取对应的值。

### 竞态

>  资源竞争，就是在程序中，同一块内存同时被多个 goroutine 访问。我们使用 go build、go run、go test 命令时，添加` -race` 标识可以检查代码中是否存在资源竞争。

解决这个问题，我们可以给资源进行加锁，让其在同一时刻只能被一个协程来操作。

- sync.Mutex
- sync.RWMutex

### 内存逃逸

##### **简单聊聊内存逃逸分析**

> 「逃逸分析」就是程序运行时内存的分配位置(栈或堆)，是由编译器来确定的

- [x] **`目的`**：决定内存分配地址是堆还是栈
- [x] 逃逸分析在编译阶段完成
- [x] 如果函数外部没有引用，则优先放在栈中
- [x] 如果函数外部有引用，则必定放在堆中
- [x] 若栈中空间不足，则必定放在堆中

##### 函数返回局部变量的指针是否安全

- [ ] 在Go里面返回局部变量的指针是安全的。因为Go会进行**逃逸分析**

##### **发生内存逃逸的经典场景**

- [x] 指针逃逸
- [x] 栈空间不足
- [x] 变量大小不确定
- [x] 动态类型
- [x] 闭包引用对象

### golang垃圾回收

##### 标记清除法

>  分为两个阶段：标记和清除

- [ ] `标记阶段`：从根对象出发寻找并标记所有存活的对象。
- [ ] `清除阶段`：遍历堆中的对象，回收未标记的对象，并加入空闲链表。

`缺点`：是需要暂停程序STW。

##### **三色标记法**

1. 栈扫描（STW），初始状态下所有对象都是白色的。
2. 从根节点开始遍历所有可达对象，把遍历到的对象变成灰色对象，放入待处理队列
3. 遍历灰色对象队列，将灰色对象引用的对象也变成灰色对象，然后将遍历过的灰色对象变成黑色对象。
4. 循环步骤3，直到灰色队列为空为止。此时所有引用对象都被标记为黑色，所有不可达的对象依然为白色，白色的就是需要进行回收的对象。
5. 通过混合写屏障检测对象变化，重复以上操作

##### **强弱三色不变式**

- [ ] `强`： 强制性的不允许黑色对象引用白色对象
- [ ] `弱`： 黑色对象可以引用白色对象，需要保证白色对象存在其他灰色对象对它的引用，或者它的链路上游存在灰色对象

##### **屏障**

- [ ] `插入写屏障`：
  * 具体操作： 在A引用B时，B被标记为灰色， 满足强三色不变式
  * 不足： 结束时需要STW来重新扫描栈
- [ ] `删除屏障`：
  * 具体操作：被删除的对象，如果自身为灰色或白色，那么被标记为灰色，满足弱三色不变式
  * 不足：回收精度低，一个对象即使被删除了最后一个指向它的指针依旧可以活过这一轮，在下一轮GC中被清理
- [ ] `混合写屏障`：
  * 具体操作：
    1. **GC开始**：优先扫描栈， 栈上的可达对象全部扫描并标记为黑色 ==(之后不再进行第二次重复扫描，无需STW)== 
    2. **GC期间**：
       * 任何在栈上创建的新对象，均为黑色
       * 被删除对象标记为灰色
       * 被添加的对象被标记为灰色 
  * 满足：变形的弱三色不变式==（结合插入、删除屏障的优点）==



##### **GC的触发条件**

- [ ] 主动触发：通过调用` runtime.GC `来触发GC，此调用阻塞式地等待当前GC运行完毕。
- [ ] 被动触发：
  * 使用`步调（Pacing）算法`，其核心思想是控制内存增长的比例,每次内存分配时检查当前内存分配量是否已达到阈值（环境变量GOGC）：默认100%，即当内存扩大一倍时启用GC
  * 使用`系统监控`，当超过两分钟没有产生任何GC时，强制触发 GC

#####  Golang的内存模型中为什么小对象多了会造成GC压力

- [ ] 通常小对象过多会导致GC三色法消耗过多的CPU。优化思路是，减少对象分配。

### Goroutine

##### 在Go函数中为什么会发生内存泄露

> Goroutine 需要维护执行用户代码的上下文信息，在运行过程中需要消耗一定的内存来保存这类信息，如果一个程序持续不断地产生新的 goroutine，且不结束已经创建的 goroutine 并复用这部分内存，就会造成内存泄漏的现象。

##### 为什么有协程泄露(Goroutine Leak)

> 协程泄漏是指协程创建之后没有得到释放。

`主要原因`有：

1. 缺少接收器，导致发送阻塞
2. 缺少发送器，导致接收阻塞
3. 死锁。多个协程由于竞争资源导致死锁。
4. WaitGroup Add()和Done()不相等，前者更大。 

##### goroutine什么情况会发生内存泄漏？如何避免



##### 如何控制协程数目

> 对于协程，可以用带缓冲区的channel来控制

下面的例子是协程数为1024的例子：

```go
var wg sync.WaitGroup
ch := make(chan struct{}, 1024)
for i:=0; i<20000; i++{
	wg.Add(1)
	ch<-struct{}{}
	go func(){
		defer wg.Done()
		<-ch
	}
}
wg.Wait()
```

### Mutex

##### mutex有几种模式

> mutex有两种模式：**normal** 和 **starvation**

正常模式

所有goroutine按照FIFO的顺序进行锁获取，被唤醒的goroutine和新请求锁的goroutine同时进行锁获取，通常**新请求锁的goroutine更容易获取锁**(持续占有cpu)，被唤醒的goroutine则不容易获取到锁。公平性：否。

饥饿模式

所有尝试获取锁的goroutine进行等待排队，**新请求锁的goroutine不会进行锁获取**(禁用自旋)，而是加入队列尾部等待获取锁。公平性：是。

### GMP

##### **GMP分别是什么，分别有多少数量**

- [ ] G（Goroutine）：即Go协程，每个go关键字都会创建一个协程。

- [ ] M（Machine）：工作线程，在Go中称为Machine，数量对应真实的CPU数（真正干活的对象）。

- [ ] P（Processor）：处理器（Go中定义的一个摡念，非CPU），包含运行Go代码的必要资源，用来调度 G 和 M 之间的关联关系，其数量可通过 `(GOMAXPROCS)` 来设置，默认为核心数。


M必须拥有P才可以执行G中的代码，P含有一个包含多个G的队列，P可以调度G交由M执行。

##### Goroutine调度策略

- 队列轮转：P 会周期性的将G调度到M中执行，执行一段时间后，保存上下文，将G放到队列尾部，然后从队列中再取出一个G进行调度。除此之外，P还会周期性的查看全局队列是否有G等待调度到M中执行。
- 系统调用：当G0即将进入系统调用时，M0将释放P，进而某个空闲的M1获取P，继续执行P队列中剩下的G。M1的来源有可能是M的缓存池，也可能是新建的。
- 当G0系统调用结束后，如果有空闲的P，则获取一个P，继续执行G0。如果没有，则将G0放入全局队列，等待被其他的P调度。然后M0将进入缓存池睡眠。

##### **全局队列**

- [ ] 存放等待运行的G

##### **P的本地队列**

- [ ] 存放等待运行的G

- [ ] 数量限制：256个G

##### **P列表**

- [ ] 程序启动时创建

- [ ] 最多有GOMAXPROCS个==(可配置)==

##### **M列表**

- [ ] 当前操作系统分配到当前Go程序的内核线程数

##### **P和M的数量问题**

P：

- [ ] 环境变量`$GOMAXPROCS`

- [ ] 在程序中通过`runtime.GOMAXPROCS()`来设置

M：

- [ ] Go语言本身限定M的最大量为10000
- [ ] 有一个M阻塞，会创建一个新的M
- [ ] 如果有M空闲，那么就会回收或者睡眠

![a1fe5504ded8050f2692d826a514f34e](D:\图\学习\a1fe5504ded8050f2692d826a514f34e.png)

##### **怎么查看Goroutine的数量**

- [ ] 在Golang中,GOMAXPROCS中控制的是未被阻塞的所有Goroutine,可以被 Multiplex 到多少个线程上运行,`通过GOMAXPROCS可以查看Goroutine的数量`

##### **怎么限制Gorountine的数量**

- [ ] `使用通道`。每次执行的go之前向通道写入值，直到通道满的时候就阻塞了

##### **Gorountine和线程的区别**

- [ ] 一个线程可以有多个协程
- [ ] 线程、进程都是同步机制，而协程是异步
- [ ] 协程可以保留上一次调用时的状态，当过程重入时，相当于进入了上一次的调用状态
- [ ] 协程是需要线程来承载运行的，所以协程并不能取代线程，`「线程是被分割的CPU资源，协程是组织好的代码流程」`

##### Go主协程如何等其余协程完再操作？

使用`sync.WaitGroup`。WaitGroup，就是用来等待一组操作完成的。WaitGroup内部实现了一个计数器，用来记录未完成的操作个数。Add()用来添加计数；Done()用来在操作结束时调用，使计数减一；Wait()用来等待所有的操作结束，即计数变为0，该函数会在计数不为0时等待，在计数为0时立即返回。

### 快问快答

##### 微服务了解吗

微服务是一种开发软件的架构和组织方法，其中软件由通过明确定义的 API 进行通信的小型独立服务组成。微服务架构使应用程序更易于扩展和更快地开发，从而加速创新并缩短新功能的上市时间。

![img](D:\图\学习\monolith_1-monolith-microservices.70b547e30e30b013051d58a93a6e35e77408a2a8.png)

#####  init() 函数是什么时候执行的，特点?

> 在main函数之前执行

- 初始化不能采用初始化表达式初始化的变量；

- 程序运行前执行注册

- 实现sync.Once功能

- 不能被其它函数调用

- init函数没有入口参数和返回值：

  - ```go
    func init(){
    	register...
    }
    ```

- 每个包可以有多个init函数，**每个源文件也可以有多个init函数**。

- 同一个包的init执行顺序，golang没有明确定义，编程时要注意程序不要依赖这个执行顺序。

- 不同包的init函数按照包导入的依赖关系决定执行顺序。

##### 请你讲一下Go面向对象是如何实现的

Go实现面向对象的两个关键是struct和interface。

封装：对于同一个包，对象对包内的文件可见；对不同的包，需要将对象以大写开头才是可见的。

继承：继承是编译时特征，在struct内加入所需要继承的类即可：

```text
type A struct{}
type B struct{
A
}
```

多态：多态是运行时特征，Go多态通过interface来实现。类型和接口是松耦合的，某个类型的实例可以赋给它所实现的任意接口类型的变量。

Go支持多重继承，就是在类型中嵌入所有必要的父类型。

##### go 打印时 %v %+v %#v 的区别？

- %v 只输出所有的值；
- %+v 先输出字段名字，再输出该字段的值；
- %#v 先输出结构体名字值，再输出结构体（字段名字+字段的值）；

```go
 a := &student{id: 1, name: "Lee"}

 fmt.Printf("a=%v \n", a) // a=&{1 Lee} 
 fmt.Printf("a=%+v \n", a) // a=&{id:1 name:Lee} 
 fmt.Printf("a=%#v \n", a) // a=&main.student{id:1, name:"Lee"}
```

##### 如何交换 2 个变量的值

```go
a,b = b,a
*a,*b = *b, *a
```

##### init() 函数是什么时候执行的

> 在main函数之前执行。

#####  空 struct{} 占用空间么

> 可以使用` unsafe.Sizeof `计算出一个数据类型实例需要占用的字节数:

- [ ] 空结构体 struct{} 实例不占据任何的内存空间。

#####  空 struct{} 的用途

>  因为空结构体不占据内存空间，因此被广泛作为各种场景下的占位符使用。

1. `将 map 作为集合(Set)使用时`，可以将值类型定义为空结构体，仅作为占位符使用即可。
2. `不发送数据的信道(channel)`
   使用 channel 不需要发送任何的数据，只用来通知子协程(goroutine)执行任务，或只用来控制协程并发调度
3. `结构体只包含方法，不包含任何的字段`

##### **究竟在什么情况下才使用指针**

- 使用指针方法能够修改接收者指向的值
- 可以避免在每次调用方法时复制该值，在值的类型为大型结构体时，这样做会更加高效

##### Go的Struct能不能比较

- 相同struct类型的可以比较
- 不同struct类型的不可以比较,编译都不过，类型不匹配

##### go 中除了加 Mutex 锁以外还有哪些方式安全读写共享变量？

> Go 中 Goroutine 可以通过 Channel 进行安全读写共享变量。

##### Go中的map如何实现顺序读取

> Go中map如果要实现顺序读取的话，可以先把map中的key，通过sort包排序。

##### **new 和 make 的区别**

- [ ] 首先我们得知道，Go的数据类型分为值类型和引用类型，其中

  值类型是 int、float、string、bool、struct和array，它们直接存储值，分配栈的内存空间，它们被函数调用完之后会释放

  引用类型是 slice、map、chan和值类型对应的指针 它们存储是一个地址（或者理解为指针）,指针指向内存中真正存储数据的首地址，内存通常在堆分配，通过GC回收

  **区别**

  - make 仅用来分配及初始化类型为 slice、map、chan 的数据。make 返回引用，即 Type。 make 分配空间后，会进行初始化。

- new 可分配任意类型的数据，根据传入的类型申请一块内存，返回指向这块内存的指针，即类型 *Type。

##### **值传递和指针传递有什么区别**

- [ ] 值传递：`会创建一个新的副本`并将其传递给所调用函数或方法 

- [ ] 指针传递：`将创建相同内存地址的新副本`

  需要改变传入参数本身的时候用指针传递，否则值传递

  另外，如果函数内部返回指针，会发生`内存逃逸`

##### **Go语言函数传参是值类型还是引用类型**

- [ ] `在Go语言中只存在值传递`，要么是值的副本，要么是指针的副本。无论是值类型的变量还是引用类型的变量亦或是指针类型的变量作为参数传递都会发生值拷贝，开辟新的内存空间



#####  Goroutine发生了泄漏如何检测

> 可以通过Go自带的工具`pprof`或者使用`Gops`去检测诊断当前在系统上运行的Go进程的占用的资源。

##### Go语言中的内存对齐了解吗

> `CPU 访问内存时，并不是逐个字节访问，而是以字长（word size）为单位访问。` 
>
> CPU 始终以字长访问内存，如果不进行内存对齐，很可能增加 CPU 访问内存的次数
>
> 合理的内存对齐可以提高内存读写的性能，并且便于实现变量操作的原子性。

##### 两个 interface 可以比较吗

- 判断类型是否一样

reflect.TypeOf(a).Kind() == reflect.TypeOf(b).Kind()

- 判断两个interface{}是否相等

reflect.DeepEqual(a, b interface{})

- 将一个interface{}赋值给另一个interface{}

reflect.ValueOf(a).Elem().Set(reflect.ValueOf(b))

##### Go中两个Nil可能不相等吗

- [ ] Go中两个Nil可能不相等。

> interface是对非接口值的封装，内部实现包含 2 个字段，类型 T 和 值 V。一个接口等于 nil，当且仅当 T 和 V 处于 unset 状态（T=nil，V is unset）
>
> - [ ] 两个接口值比较时，会先比较 T，再比较 V
> - [ ] 接口值与非接口值比较时，会先将非接口值尝试转换为接口值，再比较

```go
func main() {
 var p *int = nil
 var i interface{} = p
 fmt.Println(i == p) // true
 fmt.Println(p == nil) // true
 fmt.Println(i == nil) // false
}
```

- 例子中，将一个nil非接口值p赋值给接口i，此时,i的内部字段为(T=*int, V=nil)，i与p作比较时，将 p 转换为接口后再比较，因此 i == p，p 与 nil 比较，直接比较值，所以 p == nil。
- 但是当 i 与nil比较时，会将nil转换为接口(T=nil, V=nil),与i(T=*int, V=nil)不相等，因此 i != nil。因此 V 为 nil ，但 T 不为 nil 的接口不等于 nil。
