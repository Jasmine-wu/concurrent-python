# Python 并发编程

## 并发编程介绍

### 学习并发编程

- 加速程序的运行
- 高薪程序员必备能力

### 程序运行的 5 种并发粒度

- 单线程
- 单线程多协程
- 多线程
- 多进程
- 多机器

### 怎样选择并发技术

- 如果单机无法搞定

  - 大数据计算

- IO 密集型

  - CPU 经常在等待 IO
  - 比如网络爬虫
  - 选择 1：多协程 coroutine
  - 选择 2：多线程 threading

- CPU 密集型

  - 计算密集型，CPU 计算为主
  - 比如加密解密
  - 使用多进程 multithreading

### 线程池和进程池

- 原理：提前创建好线程/进程放在池子里，新的 task 到来可以重用这些资源，减少了新建、终止线程/进程的开销
- 池化的好处

  - 提升性能：因为减去了大量新建、终止线程的开销，重用了线程资源
  - 适用场景：适合处理突发性大量请求或需要大量线程完成任务、但实际任务处理时间较短
  - 防御功能：能有效避免系统因为创建线程过多，而导致系统负荷过大响应变慢等问题
  - 代码简洁：使用线程池的语法比自己新建线程执行线程更加简洁

## 全局解释器锁 GIL

### 任何时刻仅有一个线程在执行。

### 在多核心处理器上，使用 GIL 的解释器也只允许同一时间执行一个线程

### GIL 目的：为了解决多线程之间数据完整性和状态同步问题

### GIL 带来的问题

- 即使使用了多线程，同一时刻也只有单个线程使用 CPU，导致多核 CPU 的浪费
- GIL 只会对 CPU 密集型的程序产生影响
- 如果程序主要是在做 I/O 操作，比如处理网络连接，那么多线程技术常常是一个明智的选择

### 规避 GIL 的方法

- 规避方法 2： 使用 multiprocessing 多进程，对 CPU 密集型计算，单独启动子进程解释器去执行
- 规避方法 2： 将计算密集型的任务转移到 C 语言中，因为 C 语言比 Python 快得多，注意要在 C 语言中自己释放 GIL

## 多线程编程

### 应用于 IO 密集型计算，比如几乎所有的网络后台服务、网络爬虫

### 引入模块

- from threading import Thread

### 新建、启动、等待结束

- t=Thread(target=func, args=(100, ))
  t.start()
  t.join()

### 数据通信

- import queue
  q = queue.Queue()
  q.put(item)
  item = q.get()

### 线程安全加锁

- from threading import Lock
  lock = Lock()
  with lock: # do something

### 信号量限制并发

- from threading import Semaphore
  semaphre = Semaphore(10)
  with semaphre: # do something

### 使用线程池

- from concurrent.futures import ThreadPoolExecutor

with ThreadPoolExecutor() as executor: # 方法 1
results = executor.map(func, [1,2,3])

    # 方法2
    future = executor.submit(func, 1)
    result = future.result()

## 多进程编程

### 应用于 CPU 密集型计算，只有发现多线程编程有性能问题时，才求助于该模块

### 引入模块

- from multiprocessing import Process

### 新建、启动、等待结束

- p = Process(target=f, args=('bob',))
  p.start()
  p.join()

### 数据通信

- from multiprocessing import Queue
  q = Queue()
  q.put([42, None, 'hello'])
  item = q.get()

### 线程安全加锁

- from multiprocessing import Lock
  lock = Lock()
  with lock: # do something

### 信号量限制并发

- from multiprocessing import Semaphore
  semaphore = Semaphore(10)
  with semaphore: # do something

### 进程池

- from concurrent.futures import ProcessPoolExecutor

with ProcessPoolExecutor() as executor: # 方法 1
results = executor.map(func, [1,2,3])

    # 方法2
    future = executor.submit(func, 1)
    result = future.result()

## 多协程编程

### 异步编程的威力

- Nginx 作为 Web 服务器：
  打败了 同步阻塞服务器 Apache， 使用更少的资源支持更多的并发连接，体现更高的效率，能够支持高达 50,000 个并发连接数的响应，使用 epoll and kqueue 作为开发模型
- Redis 为什么这么快：
  处理网络请求采用单线程+使用多路 I/O 复用模型，非阻塞 IO ，避免了不必要的上下文切换和竞争条件，也不存在多进程或者多线程导致的切换而消耗 CPU，不用去考虑各种锁的问题，不存在加锁释放锁操作，没有因为可能出现死锁而导致的性能消耗；
- Node.js 的优势：
  采用事件驱动、异步编程，为网络服务而设计。其实 Javascript 的匿名函数和闭包特性非常适合事件驱动、异步编程
  Node.js 非阻塞模式的 IO 处理带来在相对低系统资源耗用下的高性能与出众的负载能力，非常适合用作依赖其它 IO 资源的中间层服务
- Go 语言的一个优势：
  Go 使用 Goroutine 和 channel 为生成协程和使用信道提供了轻量级的语法，使得编写高并发的服务端软件变得相当容易，很多情况下完全不需要考虑锁机制以及由此带来的各种问题，相比 Python 单个 Go 应用也能有效的利用多个 CPU 核，并行执行的性能好

### 异步编程的原理

- 核心原理 1：超级循环

      - 在单线程内实现并发
      - 用一个超级循环（其实就是while true）循环，里面每次轮询处理所有的task
      - 记忆口诀：

  《the one loop》
  至尊循环驭众生
  至尊循环寻众生
  至尊循环引众生
  普照众生欣欣荣

- 核心原理 2：IO 多路复用

  - 原理

    - 是一种同步 IO 模型，实现一个线程可以监视多个文件句柄；
    - 一旦某个文件句柄就绪，就能够通知应用程序进行相应的读写操作；
    - 没有文件句柄就绪时会阻塞应用程序，交出 cpu
    - 多路是指网络连接，复用指的是同一个线程

  - 3 种实现方式

    - select

      - 数据结构：bitmap
      - 最大连接数：1024
      - fd 拷贝：每次调用 select 拷贝
      - 工作效率：轮询 O(N)

    - poll

      - 数据结构：数组
      - 最大连接数：无上限
      - fd 拷贝：每次调用 poll 拷贝
      - 工作效率：轮询 O(N)

    - epool

      - 数据结构：红黑树
      - 最大连接数：无上限
      - fd 拷贝：fd 首次调用 epool_ctl 拷贝，每次调用 epoll_wait 不拷贝
      - 工作效率：回调 O(1)

### Python 官方异步库：asyncio

- 代码例子

  - import asyncio

# 获取事件循环

loop = asyncio.get_event_loop()

# 定义协程

async def myfunc(url):
await get_url(url)

# 创建 task 列表

tasks = [loop.create_task(myfunc(url)) for url in urls]

# 执行爬虫事件列表

loop.run_until_complete(asyncio.wait(tasks))

- 优点：

  - 官方库支持
  - 明确使用 asyncio、await 关键字编程，直观易读

- 缺点：

  - 很多库不支持，比如 requests

### Python 第三方异步库：Gevent

- 代码例子

  - import gevent.monkey

gevent.monkey.patch_all()

import gevent
import blog_spider
import time

begin = time.time()
for url in blog_spider.urls:
blog_spider.craw(url)
end = time.time()
print("single thread, cost = ", end - begin)

begin = time.time()
tasks = [gevent.spawn(blog_spider.craw, url) for url in blog_spider.urls]
gevent.joinall(tasks)
end = time.time()
print("gevent, cost = ", end - begin)

- 原理

  - 提供猴子补丁 MonkeyPatch 方法，通过该方法 gevent 能够 修改标准库里面大部分的阻塞式系统调用，包括 socket、ssl、threading 和  select 等模块，而变为协作式运行

- 优点

  - 只需要 monkey.patch_all()，就能自动修改阻塞为非阻塞
  - 提供了 pywsgi 异步服务器可以封装 flask

- 缺点

  - 不知道它具体 patch 了哪些库修改了哪些模块、类、函数
  - 创造了“隐式的副作用”，如果出现问题很多时候极难调试

