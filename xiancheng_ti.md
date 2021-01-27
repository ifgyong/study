## 线程相关



### 1、 GCD  执行顺序 
 <details>
  <summary>点击查看题目</font></summary>
  
  ```
  dispatch_queue_t queue=dispatch_queue_create("fgy", DISPATCH_QUEUE_SERIAL);
	NSLog(@"1");
	dispatch_async(queue, ^{
		
		NSLog(@"2");
		dispatch_async(queue, ^{
			NSLog(@"3");
		});
		NSLog(@"4");
	});
	NSLog(@"5");
  
  ```
  
#### 1. 打印顺序是什么？
  
  > 15243,原因是异步加入到串行队列中，遵循FIFO原则，所以2在4前边，最后是3，而1和5则是在主线程中同步执行。所以是15243
#### 2. 队里换成DISPATCH_QUEUE_CONCURRENT 并发队列结果呢？
> 15243,还是同步先执行，15，然后24是在主队列中执行，并且加入3任务，最后执行3任务。结果是15243.
 
</details> 

 
### 2、 GCD block 综合 
 <details>
  <summary>点击查看题目</summary>
  
  ```
  
	int a = 10;
	while (a<10) {
		dispatch_async(dispatch_get_global_queue(0, 0), ^{
			a++;
		});
	}
	NSLog(@"a=%d",a);
  ```
  
#### 1. NSlog输出什么？为什么？如何打印a的真正的值。
> 1. a++报错，因为block引用外部的变量需要__block,然后再进行赋值。
> 改完：	__block typeof(int) __blockA=a; 或者：__block int a = 10;

> 2. 改完之后输出10左右，不是一定是10，因为给全局队列添加了n次，在主线程比较a<10,但是a++任务是添加到全局队列中，从添加到执行有一定耗时，添加了次数>=10次，所以a的值不确定。

**根据队列的特性 FIFO，后添加的后执行，所以在最后添加到全局队列中获取值。**

最终修改之后的代码：

 ```
	__block int a = 0;
	while (a<10) {
		dispatch_async(dispatch_get_global_queue(0, 0), ^{
			a++;
			NSLog(@"a=%d",a);
		});
	}
	NSLog(@"主线程的值：a=%d",a);
	dispatch_async(dispatch_get_global_queue(0, 0), ^{
		NSLog(@"真正的值：a=%d",a);
	});
 ```
 
> dispatch_async 异步执行
> dispatch_sync 同步执行，阻塞当前线程
  </details>
  
  
  
### 2、 GCD block 综合 多个图片任务执行合并成一个图片 *3种*
 <details>
  <summary>点击查看题目</summary>
  
#### 1.栅栏函数
``` 
  	dispatch_queue_t queue=dispatch_queue_create("com.fygong.cn", DISPATCH_QUEUE_CONCURRENT);
	dispatch_async(queue, ^{
		NSLog(@"task1");
	});
	dispatch_async(queue, ^{
		NSLog(@"task2");
	});
	
	dispatch_async(queue, ^{
		NSLog(@"task3");
	});
	dispatch_async(queue, ^{
		NSLog(@"任务都执行完毕了");
	});

```

#### 2. 线程组 dispatch_group_t
利用dispatch_group_enter来标记当前组的任务的完成状态，用dispatch_group_leave来标记完成。来实现依赖关系。
	
```
	/*
	 A   B   C
	   D   E
		 F
	
	 D依赖于B，E依赖于C，F依赖于F，
	 A和B执行完执行D
	 B和C执行完执行E
	 D和E执行完执行F
	 **多看几遍有点难懂**
	 1. 进入g1 执行A
	 2. 进入g1 g2 执行B
	 3. 进入g3 ，监听g1的通知 执行D
	 4. 进入g3，监听g2的通知 执行E
	 5. 监听g3的通知，执行F。
	 一共分了三个组，其实就是三个依赖，每个依赖完成会通知下一个依赖。
	 */
	dispatch_group_t g1 = dispatch_group_create();
	dispatch_group_t g2 = dispatch_group_create();
	dispatch_group_t g3 = dispatch_group_create();
	
	dispatch_group_enter(g1);
	dispatch_async(dispatch_get_global_queue(0, 0), ^{
		for(int i = 0; i < 3; i++){
			NSLog(@"A");
		}
		dispatch_group_leave(g1);
	});
	
	dispatch_group_enter(g1);
	dispatch_group_enter(g2);
	dispatch_async(dispatch_get_global_queue(0, 0), ^{
		for(int i = 0; i < 3; i++){
			NSLog(@"B");
		}
		dispatch_group_leave(g1);
		dispatch_group_leave(g2);
	});
	
	dispatch_group_enter(g2);
	dispatch_async(dispatch_get_global_queue(0, 0), ^{
		for(int i = 0; i < 3; i++){
			NSLog(@"C");
		}
		dispatch_group_leave(g2);
	});
	
	dispatch_group_enter(g3);
	dispatch_group_notify(g1, dispatch_get_global_queue(0, 0), ^{
		for(int i = 0; i < 3; i++){
			NSLog(@"D");
		}
		dispatch_group_leave(g3);
	});;
	
	dispatch_group_enter(g3);
	dispatch_group_notify(g2, dispatch_get_global_queue(0, 0), ^{
		for(int i = 0; i < 3; i++){
			NSLog(@"E");
		}
		dispatch_group_leave(g3);
	});
	
	dispatch_group_notify(g3, dispatch_get_global_queue(0, 0), ^{
		for(int i = 0; i < 3; i++){
			NSLog(@"F");
		}
	});
```

#### 3. NSBlockOperation 设置依赖关系和并发数来指定执行顺序。

```
	
	NSBlockOperation *opD = [NSBlockOperation blockOperationWithBlock:^{
		
		for (int i = 0 ; i <3; i ++) {
			NSLog(@"D");
		}
	}];
	NSBlockOperation *opE = [NSBlockOperation blockOperationWithBlock:^{
		
		for (int i = 0 ; i <3; i ++) {
			NSLog(@"E");
		}
	}];
	
	
	NSBlockOperation *opF = [NSBlockOperation blockOperationWithBlock:^{
		
		for (int i = 0 ; i <3; i ++) {
			NSLog(@"F");
		}
	}];
	
	NSBlockOperation *opA = [NSBlockOperation blockOperationWithBlock:^{
		
		
		for (int i = 0 ; i <3; i ++) {
			NSLog(@"A");
		}
	}];
	
	NSBlockOperation *opB = [NSBlockOperation blockOperationWithBlock:^{
		
		for (int i = 0 ; i <3; i ++) {
			NSLog(@"B");
		}
	}];
	
	
	NSBlockOperation *opC = [NSBlockOperation blockOperationWithBlock:^{
		
		for (int i = 0 ; i <3; i ++) {
			NSLog(@"C");
		}
	}];
	
	[opD addDependency:opA];
	[opD addDependency:opB];
	
	[opE addDependency:opC];
	[opE addDependency:opB];
	
	
	[opF addDependency:opD];
	[opF addDependency:opE];
	
	NSOperationQueue *operationQueue=[[NSOperationQueue alloc]init];
	operationQueue.maxConcurrentOperationCount=5;
	// 添加之后自动开始执行任务
	[operationQueue addOperations:@[opA,opB,opC,opD,opE,opF] waitUntilFinished:NO];
```
  
</details>

### 100、 读写锁 引用对象 延迟释放问题
 <details>
  <summary>点击查看题目</font></summary>
```
+ (dispatch_queue_t )queue{
	static dispatch_once_t onceToken;
	static dispatch_queue_t queue;
	dispatch_once(&onceToken, ^{
		queue=dispatch_queue_create("test", DISPATCH_QUEUE_CONCURRENT);
	});
	return queue;
}
-(void)configLock{
	pthread_rwlockattr_t _attr;
	pthread_rwlockattr_init(&_attr);
	
	int ret = pthread_rwlock_init(&_rwlock, &_attr);
	if (ret == 0) {
		NSLog(@"初始化成功");
	}else{
		NSLog(@"初始化失败");
	}
}
-(void)write2{
	
	dispatch_queue_t queue = [RWLock queue];
	dispatch_async(queue, ^{
		int retLock = pthread_rwlock_wrlock(&self->_rwlock);
		if (retLock == 0) {
			[self write];
			pthread_rwlock_unlock(&self->_rwlock);
		}else{
			NSLog(@"读操作加锁失败 code:%d ",retLock);
		}
	});
}
- (void)readBlock2{
	
	dispatch_queue_t queue = [RWLock queue];
	dispatch_async(queue, ^{
		int retLock = pthread_rwlock_rdlock(&self->_rwlock);
		if (retLock == 0) {
			[self read];
			pthread_rwlock_unlock(&self->_rwlock);
		}else{
			NSLog(@"读操作加锁失败 code:%d ",retLock);
		}
	});
}
-(void)read{
	NSLog(@"read start");
	sleep(2);
	NSLog(@"read end");
}
-(void)write{
	
	NSLog(@"write start");
	sleep(2);
	NSLog(@"write end");
}
-(void)dealloc{
	pthread_rwlock_destroy(&_rwlock);
	NSLog(@"dealloc %s",__func__);
}
```

#### 在A页面进行读写操作，读写操作完成之前返回了上级页面，读写操作会中断吗？为什么？

> **不会中断，因为block引用了obj，obj被gcd引用，所以会执行完block然后再销毁obj对象。**

#### 除了读写锁还有其他的实现方式吗？
> 并发队列+栅栏函数，并发队列不能是全局队列，因为栅栏函数无法拦截全局队列。否则所有在全局函数中的任务都会被拦截。


 </details>
