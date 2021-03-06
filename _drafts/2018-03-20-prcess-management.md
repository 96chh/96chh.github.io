---
layout: post
title:  '进程管理'
categories: Computer
author: CHH
---

* content
{:toc}




## 一、进程与线程

### 1. 进程

进程是资源分配的基本单位。

PCB（进程控制块，Process Control Block）描述进程的基本信息和运行状态。

创建进程、撤销进程都是对 PCB 进行操作。

进程可以并发地执行。

![多进程并发执行](https://upload-images.jianshu.io/upload_images/5690299-156e8b0ac2d5a380.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2. 线程

线程是独立调度的基本单位。

### 3. 联系与区别

- 拥有资源：一个进程可以有多个线程，它们共享进程资源。这也说明了资源的分配以进程为单位。
- 调度：从一个进程内的线程切换到另一个进程中的线程时，会引起进程切换。这也说明了线程是独立调度的基本单位。
- 系统开销：**进程创建或撤销**时，系统都要为之分配或回收**资源**，如内存空间、I/O 设备等，开销远大于创建或撤销线程时的开销。
**进程切换**时，当前进程 **CPU 环境**的保存、新调度进程 CPU 环境的设置，而线程切换时只需保存和设置少量**寄存器**内容，开销很小。
- 通信方面：
进程间通信 (IPC) 需要进程**同步和互斥**手段的辅助，以保证数据的一致性。
线程间通信可以通过直接读/写同一进程中的**数据段**（如全局变量）。

## 二、进程状态切换

- 就绪状态（ready）：等待 CPU 时间
- 运行状态（running）
- 阻塞状态（waiting）：等待资源

**说明：**

- 只有就绪态和运行态可以相互转换，其它的都是单向转换。
- 就绪状态的进程，通过调度算法从而获得 CPU 时间片，转为运行状态；
运行状态的进程，用完 CPU 时间片后就会转为就绪状态，等待下一次调度。
- 阻塞状态是从运行状态由于缺少需要的资源转换而来，最终转换为就绪状态。

![进程状态](https://upload-images.jianshu.io/upload_images/5690299-5a65dc5cb94ba19c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 三、调度算法

针对不同环境来讨论，分三种：批处理、交互式、实时系统

### 1. 批处理系统地调度

- 先来先服务（FCFS，first-come first-serverd）：短作业有可能相对等待时间过长。
- 短作业优先（SJF，shortest job first）：长作业有可能饿死。
- 最短剩余时间优先（SRTN，shortest remaining time next）

### 2. 交互式系统调度

#### 2.1. 优先级调度

一是手动赋予优先权；
二是把**响应比**作为优先权。
响应比 = 响应时间 / 要求服务时间 = (等待时间 + 要求服务时间) / 要求服务时间
这就解决「短作业优先调度算法」长作业可能会饿死的问题。因为等待时间 ↑，响应比 ↑。

#### 2.2. 时间片轮转

在 FCFS 基础上，队列首进程执行一个时间片，当时间片用完时，将它送往就绪队列的末尾，同时继续把 CPU 时间分配给队首的进程。

时间片轮转算法的效率和时间片的大小有很大关系。因为进程切换都要保存进程的信息并且载入新进程的信息，如果时间片太小，会导致进程切换得太频繁，在进程切换上就会花过多时间。

#### 2.3. 多级反馈队列

多级队列是为需要连续执行多个时间片的进程考虑。
每个队列时间片大小都不同，例如 1,2,4,8,..。进程在第一个队列没执行完，就会被移到下一个队列。
进程要求服务时间越长，最后移到的队列的时间片越就越长。
当然，时间片小的队列比时间片大的队列优先级要高。
实质是对「FCFS、时间片轮转」的优化，减少进程的切换次数。

### 3. 实时系统调度

实时系统要求一个服务请求在一个确定时间内得到响应。
分为硬实时和软实时，硬实时必须满足绝对的截止时间，软实时可以容忍一定的超时。

### 4. 总结

>在「先来先服务」的基础上，加了 CPU 时间片，就变成了「时间片轮转」，为了减少进程切换次数，引进了「多级队列」。

>在「短队列优先」的基础上，为了解决长队列有可能被饿死，引进了「响应比优先级调度」。

## 四、进程同步

### 1. 临界区

是一段代码，这段代码是对临界资源进行访问。
为了互斥访问临界资源，每个进程在进入临界区之前，需要先进行检查。

```html
entry section
critical section;    // 关键段
exit section         
```

### 2. 同步与互斥

同步：多个进程按照一定顺序执行。
互斥：同一时刻只能有一个进程进入临界区。

### 3. 信号量

Semaphore，是一个整型变量，可以对其进行 up、down 操作。
up 和 down 操作需要被设计成「原语」。可以用屏蔽中断实现。
>down：减一，减到 0 后进入睡眠。
up：加一，唤醒睡眠。

**互斥量（Mutex）**：信号量只能取 0 或 1。0 表示临界区已经加锁，1 表示临界区解锁。

应用：使用信号量解决 生产者-消费者问题

问题描述：使用一个缓冲区来保存物品，只有缓冲区没有满，生产者才可以放入物品；只有缓冲区不为空，消费者才可以拿走物品。

思路：
（1）缓冲区属于临界资源，因此需要使用一个互斥量 mutex 来控制对缓冲区的互斥访问。
（2）empty 表示空缓冲区，full 表示满缓冲区。当 empty 不为 0 时，生产者才可以放入物品；当 full 不为 0 时，消费者才可以取走物品。

注意事项：
不能先对缓冲区进行加锁，再测试信号量。也就是说，不能先执行 down(mutex) 再执行 down(empty)。
生产者对缓冲区加锁后，执行 down(empty) 操作，发现 empty = 0，此时生产者睡眠。
由于缓冲区已加锁，消费者无法执行 up(empty) 操作，empty 永远都为 0，那么生产者和消费者就会一直等待下去，造成死锁。

```c
# define N 100
typedef int semaphore;
semaphore mutex = 1;
semaphore empty = N;
semaphore full = 0;

void producer() {
    while(TRUE){
        int item = produce_item();
        down(&empty);
        down(&mutex);
        insert_item(item);
        up(&mutex);
        up(&full);
    }
}

void consumer() {
    while(TRUE){
        down(&full);
        down(&mutex);
        int item = remove_item();
        up(&mutex);
        up(&empty);
        consume_item(item);
    }
}
```

---

参考文档：

[https://github.com/CyC2018/Interview-Notebook/blob/master/notes/%E8%AE%A1%E7%AE%97%E6%9C%BA%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F.md#%E4%BA%8C%E8%BF%9B%E7%A8%8B%E7%AE%A1%E7%90%86](https://github.com/CyC2018/Interview-Notebook/blob/master/notes/%E8%AE%A1%E7%AE%97%E6%9C%BA%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F.md#%E4%BA%8C%E8%BF%9B%E7%A8%8B%E7%AE%A1%E7%90%86)