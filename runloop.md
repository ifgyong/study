# Runloop
### 1. 线程和Runloop的关系
<details>
  <summary>点击查看详细内容</summary>
  
  -  `runloop`和线程是一一对应的，一个`runloop`对应一个核心线程，为什么或是核心的，因为`runloop`是可以嵌套的，但是核心只能有一个，他们的关系保存在一个全局的字典里。
  -  `runloop`是来管理线程的，当线程的`runloop`被开启后，线程会在执行完任务后进入休眠状态，有了任务就会被唤醒去执行任务。
  -  `runloop`在第一次被创建，在线程结束时销毁。
  -  对于主线程来说，`runloop`在程序启动就创建好了。
  -  对于子线程来说，`runloop`是懒加载的，只有当我们使用时才会被创建，所以子线程用定时器要注意，确保线程的`runloop`被创建，不然定时器不会回调。
  
  - `Source0`事件：非基于`Port`的处理事件，不能主动唤醒休眠中的`RunLoop`，需要手动触发。我们触摸屏幕时，先摸到硬件(屏幕)，屏幕表面的事件会先包装成`Event`，`Event`先告诉`source1``（mach_port）`，`source1`唤醒`RunLoop`， 然后将事件`Event`分发给`source0`，然后由`source0`来处理。

- `Source1`事件：基于`mach_Port`的，来自系统内核或者其他进程或线程的事件，可以主动唤醒休眠中的`RunLoop`

  </details>
  