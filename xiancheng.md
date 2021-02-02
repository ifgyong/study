

### 1. 线程之间通讯
<details>
  <summary>点击查看详细内容</summary>
   线程之间通过NSPort通讯，将NSPort加入到runloop中，可以通过设置port的代理来监听该端口接受的数据，其实是先讲数据发给runloop，然后经过runloop中转到delegate去处理数据。
   
[查看具体代码]((https://github.com/ifgyong/demo/tree/master/OC/%E7%BA%BF%E7%A8%8B%E9%80%9A%E8%AE%AF))  
</details>  

### 2.子线程生命周期控制
<details>
  <summary>点击查看详细内容</summary>
```
  __weak typeof(self) __weakSelf = self;
    self.thread = [[FYThread alloc]initWithBlock:^{
        [[NSRunLoop currentRunLoop] addPort:[NSPort port] forMode:NSDefaultRunLoopMode];
        NSLog(@"%@",[NSThread currentThread]);
        NSLog(@"--start--");
        __weakSelf.shouldKeepRunning = YES;//默认运行
        NSRunLoop *theRL = [NSRunLoop currentRunLoop];
        while (__weakSelf && __weakSelf.shouldKeepRunning  ){
            [theRL runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]];
        };
        NSLog(@"--end--");
    }];
```
</details>