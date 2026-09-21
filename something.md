## 语法 
### c#8
    Activity.Current is { } activity ? Convert.FromHexString(activity.TraceId.ToHexString()) : null;
### BlockingCollection
    BlockingCollection<T> 是 .NET 中专门为 生产者-消费者模式（Producer-Consumer Pattern） 设计的高级线程安全集合类。它极大地简化了多线程之间的数据共享与同步，让开发者无需手动管理锁（lock）和信号量（SemaphoreSlim 等）。
    一、 核心特性
        1、自带阻塞机制：读阻塞：当集合为空时，消费者调用 Take() 会自动挂起，直到生产者加入新数据才会被唤醒。写阻塞：支持设置容量上限（Bounded Capacity）。当集合满了，生产者调用 Add() 会自动挂起，直到消费者消费了数据腾出空间。
        2、线程安全：并发环境下多个线程同时执行添加（Add）或删除（Take）操作无需加锁。
        3、支持生命周期管理：可以通过调用 CompleteAdding() 声明生产结束，防止消费者无限期死等。
        4、可灵活替换底层结构：默认基于先进先出的 ConcurrentQueue<T> 队列，但也支持通过构造函数更换为后进先出的 ConcurrentStack<T> 或无序的 ConcurrentBag<T>。


### wpf线程
    _synchronizationContext = SynchronizationContext.Current; 用于保存创建 KeyLoggerViewModel 时的线程上下文，它主要解决后台反馈流更新 UI 时的线程切换问题。


## 安全
 ### 免杀 
     https://www.cnblogs.com/mykr3/p/17905122.html
 ### BypassUAC
     https://stack.chaitin.com/techblog/detail/43

## 技术
 ### 虚拟多屏
    https://cloud.tencent.com/developer/article/2428215