[TOC]

  

# Java

  

## Java集合框架

  

![Java集合框架](https://oss.javaguide.cn/github/javaguide/java/collection/java-collection-hierarchy.png)

  

---

  

## Java内存模型（JMM）

  

![Java内存模型](https://oss.javaguide.cn/github/javaguide/java/concurrent/jmm.png)

  

---

  

## JVM内存模型

  

![JVM内存模型](https://oss.javaguide.cn/github/javaguide/java/jvm/java-runtime-data-areas-jdk1.8.png)

  

---

  

## Java垃圾回收（GC）

  

### Java堆空间结构

  

年龄计数器：每个对象初始年龄为1岁，经过一次MinorGC增加1岁，15岁（默认）进入老年代

  

1. `新生代（Young Generation）`：大多数对象在新生代的`Eden`区分配，当`Eden`区空间不足，虚拟机触发一次`Minor GC`

2. `老生代（Old Generation）`：大对象（`字符串`、`数组`）直接存入老年代，长期存活的对象也进入老年代

3. `永生代（Permanent Generation）`：在`JDK8`后被`元空间（Metaspace）`取代并使用`直接内存`

  

![Java堆空间结构](https://oss.javaguide.cn/github/javaguide/java/jvm/hotspot-heap-structure.png)

  

### 如何判断对象是否死亡（两种方法）

1. `引用计数法`：为每个对象添加一个引用计数器，计数器为0的对象认为不再使用。本方法实现简单、效率高，但主流虚拟机不使用，因为难以解决对象之间`循环引用`的问题。

2. `可达性分析`：从一系列`GC Roots`对象（）作为起点，从这些节点向下搜索，当一个对象没有任何引用链接相连，则该对象不再使用，需要回收。

  

![可达性分析算法](https://oss.javaguide.cn/github/javaguide/java/jvm/jvm-gc-roots.png)

  

#### 哪些对象可以作为 GC Roots 呢？

- 虚拟机栈(栈帧中的局部变量表)中引用的对象

- 本地方法栈(Native 方法)中引用的对象

- 方法区中类静态属性引用的对象

- 方法区中常量引用的对象所有被同步锁持有的对象

- JNI（Java Native Interface）引用的对象

  

---

  

### 四种引用（引用强度逐渐减弱）

1. `强引用（StongReference）`：以前我们使用的大部分引用实际上都是强引用，这是使用最普遍的引用。如果一个对象具有强引用，那就类似于必不可少的生活用品，垃圾回收器绝不会回收它。当内存空间不足，Java 虚拟机宁愿抛出 OutOfMemoryError 错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足问题。

2. `软引用（SoftReference）`：如果内存空间足够，垃圾回收器就不会回收它，如果内存空间不足了，就会回收这些对象的内存。

3. `弱引用（WeakReference）`：一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程， 因此不一定会很快发现那些只具有弱引用的对象。

4. `虚引用（PhantomReference）`：和没有任何引用一样，在任何时候都可能被垃圾回收。主要用来跟踪对象被垃圾回收的活动。

  

### 垃圾回收算法

1. 标记-清除

标记-清除（Mark-and-Sweep）算法分为“标记（Mark）”和“清除（Sweep）”阶段：首先标记出所有不需要回收的对象，在标记完成后统一回收掉所有没有被标记的对象。

2. 复制算法

为了解决标记-清除算法的效率和内存碎片问题，复制（Copying）收集算法出现了。它可以将内存分为大小相同的两块，每次使用其中的一块。当这一块的内存使用完后，就将还存活的对象复制到另一块去，然后再把使用的空间一次清理掉。这样就使每次的内存回收都是对内存区间的一半进行回收。

3. 标记-整理算法

标记-整理（Mark-and-Compact）算法是根据老年代的特点提出的一种标记算法，标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象回收，而是让所有存活的对象向一端移动，然后直接清理掉端边界以外的内存。

4. 分代收集算法

当前虚拟机的垃圾收集都采用分代收集算法，这种算法没有什么新的思想，只是根据对象存活周期的不同将内存分为几块。一般将 Java 堆分为新生代和老年代，这样我们就可以根据各个年代的特点选择合适的垃圾收集算法。比如在新生代中，每次收集都会有大量对象死去，所以可以选择”标记-复制“算法，只需要付出少量对象的复制成本就可以完成每次垃圾收集。而老年代的对象存活几率是比较高的，而且没有额外的空间对它进行分配担保，所以我们必须选择“标记-清除”或“标记-整理”算法进行垃圾收集。

  

### 垃圾收集器

1. Parallel Scavenge 收集器

```text

新生代采用标记-复制算法，老年代采用标记-整理算法

Parallel Scavenge 收集器关注点是吞吐量（高效率的利用 CPU）。CMS 等垃圾收集器的关注点更多的是用户线程的停顿时间（提高用户体验）。

JDK1.8 默认使用的是 Parallel Scavenge + Parallel Old

```

2. Parallel Old 收集器

```text

Parallel Scavenge 收集器的老年代版本。

```

3. CMS 收集器

```text

CMS（Concurrent Mark Sweep）收集器是一种以获取最短回收停顿时间为目标的收集器。它非常符合在注重用户体验的应用上使用。

CMS（Concurrent Mark Sweep）收集器是 HotSpot 虚拟机第一款真正意义上的并发收集器，它第一次实现了让垃圾收集线程与用户线程（基本上）同时工作。

```

4. G1 收集器

```text

G1 (Garbage-First) 是一款面向服务器的垃圾收集器,主要针对配备多颗处理器及大容量内存的机器. 以极高概率满足 GC 停顿时间要求的同时,还具备高吞吐量性能特征.

G1 收集器的运作大致分为以下几个步骤：

初始标记

并发标记

最终标记

筛选回收

从 JDK9 开始，G1 垃圾收集器成为了默认的垃圾收集器。

```

  

---

  

## `类加载机制`

  

将`.class`文件加载到内存中生成`java.lang.Class`类型的对象

1. `加载（Loading）`：将Java类的`字节码`文件加载到内存中，在内存中构建Java类模板对象

2. `链接（Linking）`：

   - `验证（Verification）`：确保加载类符合jvm规范，没有安全性问题

   - `准备（Preparation）`：为类的`静态变量`分配内存并初始化默认值（int：0等）

   - `解析（Resolution）`：将常量池中的符号引用转换为直接引用

3. `初始化（Initialization）`：执行构造函数，执行`static`代码块为静态变量赋值

4. `使用（Using）`：

5. `卸载（Unloading）`：

  

整个过程是由`类加载器（Class Loaders）`完成，Java的类加载器：

1. `启动类加载器（Bootstrap ClassLoader）`：

2. `扩展类加载器（Extension ClassLoader）`：

3. `应用类加载器（Application ClassLoader）`：

4. `自定义类加载器（User ClassLoader）`：

  

### `双亲委派模型（Parents Delegation Model）`

  

```text

按照类加载器的层级关系，将类的查询和加载传递给父加载器执行，如果父加载器无法完成，则尝试自己加载

```

1. `安全性`：保证了类加载器的层级关系，保证核心库的加载是由`启动类加载器`完成，保证自己写的类无法覆盖核心库类

2. `有序性`：可以避免重复加载导致程序混乱

  

`双亲委派机制`可以被打破（`Tomcat`、`SPI`），只需要自定义`ClassLoader`，重写`loadClass`方法，不依次传递给父加载器进行加载，即可打破

  

---

  

## `equals and hashcode`

  
  

## `volatile`

  

1. 可以保证在多线程环境下共享变量的`可见性`（总线锁、缓存锁MESI一致性协议、Happens-Before原则）

2. 可以通过增加内存屏障防止多个指令之间的重排序

  
  

## `JUC`

  

### `ThreadPoolExecutor`

前五个为必要参数，后两个为可选参数

1. `corePoolSize`：

2. `maximumPoolSize`：

3. `keepAliveTime`：

4. `unit`：`TimeUnit`类型

5. `workQueue`：`BlockingQueue<Runnable>`类型

6. `threadFactory`：`ThreadFactory`类型

7. `handler`：`RejectedExecutionHandler`类型，默认为`AbortPolicy`

  

---

  

# Redis

  

1. `string`：使用`简单动态字符串（SDS）`实现，

``` c

struct sdshdr {  

    int len;   // buf 中已占用空间的长度  

    int free;     // buf 中剩余可用空间的长度

    char buf[];   // 数据空间  

};

```

2. `hash`：使用字典实现，解决哈希冲突方式为拉链法

```c

typedef struct dictht {

   dictEntry **table;      //哈希表数组  

   unsigned long size;     //哈希表大小  

   unsigned long sizemask; //哈希表大小掩码，用于计算索引值

   unsigned long used;     //该哈希表已有节点的数量

}

typeof struct dictEntry {  

   void *key;  //键

   union{      //不同键对应的值的类型可能不同，使用union来处理这个问题

      void *val;

      uint64_tu64;

      int64_ts64;

   }

   struct dictEntry *next;

}

```

3. `list`：使用双向链表实现

4. `set`：

5. `zset`：有序集合，数据较少时使用`ziplist`，数据较多时使用`skiplist`

```text

对于单链表来说，即使数据是已经排好序的，想要查询其中的一个数据，只能从头开始遍历链表，这样效率很低，时间复杂度很高，是 O(n)。

那我们有没有什么办法来提高查询的效率呢？我们可以为链表建立一个“索引”，这样查找起来就会更快，如下图所示，我们在原始链表的基础上，每两个结点提取一个结点建立索引，我们把抽取出来的结点叫做索引层或者索引，down 表示指向原始链表结点的指针。

对于非常大数据量，可以增加索引层数

```

![skiplist](https://upload-images.jianshu.io/upload_images/944288-410008cc643f0248.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

  

![多层索引跳表](https://upload-images.jianshu.io/upload_images/944288-d5aa2836a06cd6e6.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

  

---

  
  


  

# Linux

  

## 常用命令

  

### tldr

```text

人生苦短，我用tldr。

tldr = Too Long Don't Read，可以帮助快速查看一些命令的使用方法（在线+缓存）

```

`tldr <command>`：会展示该命令的常用方式以及参数

  

---

  

### ranger

  

### tmux `窗口`>`窗格`

`tmux ls`

`tmux new -s <session-name | session-num>`：新建一个终端

`tmux detach`

`tmux attach -t <name>`

`tmux kill-session -t <name>`

`tmux switch`

#### tmux快捷键`Ctrl + b`（在tmux窗口中）

   - `<arrow key>`：使用方向键切换选定窗格

   -  `Ctrl + <arrow key>`：使用方向键调整窗格大小

   - `d`：分离当前会话

   - `s`：列出所有会话

   - `$`：重命名当前会话

   - `?`：帮助

   - `%`: 左右划分

   - `"`: 上下划分

   - `q`: 暂时显示窗格编号

   - `x`: 关闭当前窗口

   - `o`：切换到下一个窗格（循环）

   - `;`: 切换到上一个窗格（只能在两个中来回切换，不是在所有窗口循环切换）

   - `z`: 当前窗格全屏显示，再使用一次会变回原来大小

  

### pacman、yay

`pacman`是`archlinux`的默认包管理器

`yay`是进阶版`pacman`，推荐使用`yay`

  

---

  

### `fzf`

`fuzzy finder`：`模糊查找`神器，可以查找的不只有文件

  

### `systemctl`、`journalctl`、`crontab`

记得使用`systemctl status cronie.service`查看`crontab`定时任务是否启用

查看`crontab`日志：`journalctl -u cronie.service -n 50`：查看`cronie.service`日志后`100`行

- `crontab -l`：显示当前设置的定时任务（基于用户）

- `crontab -e`：使用编辑器编辑定时任务（基于用户）

  

### `less`、`more`

  

### `cat`、`grep`

https://www.runoob.com/linux/linux-comm-grep.html

```text

Linux grep (global regular expression) 命令用于查找文件里符合条件的字符串或正则表达式。

  

grep 指令用于查找内容包含指定的范本样式的文件，如果发现某文件的内容符合所指定的范本样式，预设 grep 指令会把含有范本样式的那一列显示出来。若不指定任何文件名称，或是所给予的文件名为 -，则 grep 指令会从标准输入设备读取数据。

```

  

`grep <search_content> <filename>`

  

`-n | --line-number`：显示行号

`-A <行数> | --after-context=<行数>`：显示匹配内容`后<行数>`行

`-B <行数> | --before-context=<行数>`：显示匹配内容前`前<行数>`行

`-C <行数> | --context=<行数>`：显示匹配内容`前后<行数>`行

  

---

  

### `ls`、`pwd`、`cd`、`z（zsh-plugin）`、`rm -i`

  

### `ln`、`ln -s`

目录无法创建硬链接，但是其有`.`和`..`两个硬链接

  

### `curl`、`wget`、

`curl`常用参数：

   - `-X`：指定请求类型，如`curl -X GET`，或`POST`等

   - `-k`等同于`--insecure`：(SSL)设置此选项将允许使用无证书的不安全SSL进行连接和传输

   - `-L`等同于`--location`：(HTTP/HTTPS)追随http响应头“Location：”定向到跳转后的页面

   - `-v`等同于`--verbose`：显示更详细的信息，调试时使用

   - `-H`等同于`--header`：(HTTP)添加一个http header(http请求头)，`-H "name:value"`

  

### `make`

  

### `asdf`

  

### `yq`

`yq eval -i '.dns.listen = "0.0.0.0:53"' /etc/clash/config.yaml `：向yaml文件修改字段（不存在则新建）,`-i`表示原地修改文件，没有的话则输出到控制台

  

### `tr`

  

## `vim`

![vi/vim键盘图](./pictures/vim.png)

  

- `G`：跳转至结尾

- `o`：新增一行

  

---

  

## Shell

https://www.runoob.com/linux/linux-shell-process-control.html

  

---

  

# 计算机网络

  

## OSI七层模型

  

1. `应用层`：为计算机用户提供服务

- `HTTP`：

- `SMTP`：

- `FTP`：

- `DNS`：

- `DHCP`：

- `Telnet`：

2. `表示层`：数据处理（编解码、加密解密、压缩解压缩）

- `SSL/TLS`：

- `JPEG`：

- `ASCII`：

3. `会话层`：管理（建立、维护、重连）应用程序之间的会话

- `RPC`：

- `PPTP`：

- `SMB`：

4. `传输层`：为两台主机进程之间的通信提供通用的数据传输服务

- `TCP`：

- `UDP`：

- `SCTP`：

5. `网络层`：路由和寻址（决定数据在网络的有走路径）

- `IP`：

- `ICMP`：

- `OSPF`：

- `RIP`：

- `BGP`：

6. `数据链路层（Data Link Layer）`：帧编码和误差纠正控制

- `ARP`：

- `Ethernet`：

- `PPP`：

- `MAC`：

7. `物理层（Physical Layer）`：透明地传输比特流传输

- `USB`：

- `IEEE 802.11（Wi-Fi）`：

- `Ethernet`：

- `Bluetooth`：

  

## TCP/IP四层模型

  

1. `应用层`：相当于OSI七层模型的`应用层`+`表示层`+`会话层`

2. `传输层`：相当于OSI七层模型的`传输层`

3. `网络层`：相当于OSI七层模型的`网络层`

4. `网络接口层`：相当于OSI七层模型的`数据链路层`+`物理层`

  

## TLS协议

  

原理：

   1.client向server发送建立连接请求，同时发送（TLS版本，加密套件，`第1随机数`）

   2.server收到后向client发送（TLS版本，加密套件，`第2随机数`），并依次发送证书、公钥、发送结束

   3.client收到后向server发送使用`公钥加密后`的`第3随机数（预主密钥）`

   4.server收到后使用私钥解密得到解密预主密钥，此时server和client均使用`第1随机数+第2随机数+预主密钥`计算得到会话密钥，此后的数据传输使用该会话密钥进行加密（对称加密）

  

---

  

# 操作系统

  

## 进程与线程

- `进程（Process）`是计算机中正在运行的程序实例，是系统运行程序的基本单位，

- `线程（Thread）`是一个比进程更小的执行单位，有自己的`程序计数器`、`虚拟机栈`和`本地方法栈`

  

### 进程间通信的方式

1. `管道/匿名管道（Pipes）`：用于具有亲缘关系的父子进程间或者兄弟进程之间的通信。

2. `有名管道（Named Pipes）`：可以实现本机任意两个进程通信。

3. `信号量（Semaphore）`：

4. `信号（Signal）`：如`kill -9 <ProcessID>`

5. `共享内存（Shared Memory）`：

6. `消息队列（Message Queuing）`：

7. `套接字（Sockets）`：主要用于在客户端和服务器之间通过网络进行通信。

  

### 线程间同步的方式

1. `互斥锁（Mutex）`：如`synchronized`

2. `读写锁（Read-Write Lock）`：

3. `信号量（Semaphore）`：

4. `屏障（Barrier）`：如`CyclicBarrier`

5. `事件（Event）`：如`Wait/Notify`

  

### 僵尸进程与孤儿进程

- `僵尸进程`：子进程已经终止，但是其父进程仍在运行，且父进程没有调用 wait()或 waitpid()等系统调用来获取子进程的状态信息，释放子进程占用的资源，导致子进程的 PCB 依然存在于系统中，但无法被进一步使用。这种情况下，子进程被称为“僵尸进程”。

- `孤儿进程`：一个进程的父进程已经终止或者不存在，但是该进程仍在运行。这种情况下，该进程就是孤儿进程。孤儿进程通常是由于父进程意外终止或未及时调用 wait()或 waitpid()等系统调用来回收子进程导致的。

  
  

# 高可用

  

## 负载均衡（Load Balance）策略

  

```text

负载均衡分为服务器端负载均衡和客户端负载均衡。

服务器端负载均衡指的是存放在服务器端的负载均衡器，例如 Nginx、HAProxy、F5 等。

客户端负载均衡指的是嵌套在客户端的负载均衡器，例如 Ribbon。

```

  

- `轮询（Round Robin）`：轮询策略按照顺序将每个新的请求分发给后端服务器，依次循环。这是一种最简单的负载均衡策略，适用于后端服务器的性能相近，且每个请求的处理时间大致相同的情况。

- `随机选择（Random）`：随机选择策略随机选择一个后端服务器来处理每个新的请求。这种策略适用于后端服务器性能相似，且每个请求的处理时间相近的情况，但不保证请求的分发是均匀的。

- `最少连接（Least Connections）`：最少连接策略将请求分发给当前连接数最少的后端服务器。这可以确保负载均衡在后端服务器的连接负载上均衡，但需要维护连接计数。

- `IP哈希（IP Hash）`：IP 哈希策略使用客户端的 IP 地址来计算哈希值，然后将请求发送到与哈希值对应的后端服务器。这种策略可用于确保来自同一客户端的请求都被发送到同一台后端服务器，适用于需要会话保持的情况。

- `加权轮询（Weighted Round Robin）`：加权轮询策略给每个后端服务器分配一个权重值，然后按照权重值比例来分发请求。这可以用来处理后端服务器性能不均衡的情况，将更多的请求分发给性能更高的服务器。

- `加权随机选择（Weighted Random）`：加权随机选择策略与加权轮询类似，但是按照权重值来随机选择后端服务器。这也可以用来处理后端服务器性能不均衡的情况，但是分发更随机。

- `最短响应时间（Least Response Time）`：最短响应时间策略会测量每个后端服务器的响应时间，并将请求发送到响应时间最短的服务器。这种策略可以确保客户端获得最快的响应，适用于要求低延迟的应用。

  
  

## 限流（Rate Limit）策略

  

- `固定窗口`：使用固定时间窗口（每分钟、每小时）来限制请求数量。

- `滑动窗口`：将时间单位划分为多个区间，每个区间有多个计数器，限制一定区间段的请求数。

- `漏桶算法（Leaky Bucket）`：请求进入一个“漏桶”，并以恒定速率流出，桶满后新的请求被丢弃或拒绝。

- `令牌桶（Token Bucket）`：有一个持续填充的令牌桶，每个请求都需要一个令牌，桶空则新的请求被拒绝或延迟。

  

## Kafka

  

```text

Kafka是一个分布式、基于发布/订阅模式的消息队列（Message Queue），主要应用于大数据实时处理领域。

发布/订阅模式：消息的发布者不会将消息直接发送给特定的订阅者，二十将发布的消息分为不同的类别，订阅者只接收感兴趣的消息。

Kafka是一个开源的分布式事件流平台（Event Streaming Platform）。

```

  

大数据场景：Kafka

Java开发：RocketMQ、ActiveMQ、RabbitMQ

  
  

## VSCode

  

### 快捷键

  

- `F5`: 启动

  
  

## GO

  

`go mod tidy`:

  
  

## 项目

  

### 人工智能模型训练平台

  

```text

  

```