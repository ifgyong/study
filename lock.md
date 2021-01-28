# Lock
### 1. dispatch_async 使用
 <details>
  <summary>点击查看详细内容</summary>

```
__block NSObject *obj =[NSObject new];
for (int i =0; i <100000; i++) {
	dispatch_async(dispatch_get_global_queue(0, 0), ^{
		@synchronized (obj) {
			obj=[NSObject new];
		}
	});
}
```

</details>

#### 2. 会有什么问题？
`synchronized`是以obj地址为存储在全局的`hashTable`确定一个锁的，当多线程都访问`[NSObject new]`会触发多个`release`，导致`obj=nil`,那么这个锁值变了，线程不再安全，应该还一个不会变的值，比如`self`,或者使用`NSLock`等其他锁。

### 3. @synchronized

自旋锁 递归锁，底层是

### 4. NSRecursiveLock 递归锁


### 5. NSLock 非递归

### 6. NSConditionLock 条件锁

### 7. dispatch_semaphore_t 信号量