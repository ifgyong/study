# Block


## 1. Block 线程 runloop综合题
 <details>
  <summary>点击查看详细内容</summary>
  
  ```
 	///1. 循环引用 self->block->self
	[self.block addBlock:^{
		self.view.backgroundColor=[UIColor redColor];
	}];
	
	///Block Class method
	// self->block->self  导致了 循环引用
-(void)addBlock:(dispatch_block_t)block{
	self.block = block;
	dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
		if (self.block) {
			self.block();
		}
	});
}


	
	//2.  不会循环引用 block->self
-(void)addBlock2:(dispatch_block_t) block{
	[[Block share] addBlock:^{
		self.view.backgroundColor=[UIColor redColor];
	}];
}


/// 3 不会循环引用 self->block gcd -> self 没有形成环。

//Block class
// 没有形成循环链 不会内存泄露
-(void)addBlockSecond:(dispatch_block_t)block{
	dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
		if (block) {
			block();
		}
	});
}
-(void)addBlock2{
	[self.block  addBlockSecond:^{
		self.view.backgroundColor=[UIColor redColor];
	}];
}
	

	

/// 4. 线程不安全 需要加读写锁 或者普通的锁都行
/// 会读写失败 原因是 多线程操作数据属于读写需要加锁，否则数据扩容的时候再去写入和读会报错
/// error for object 0x600001a8c4e0: pointer being freed was not allocated
///[NSObject new] 会执行多个release 导致 崩溃
/// 只需要将lock打开加锁即可。
- (void)test4{
	NSMutableArray *ret =[NSMutableArray new];
	NSLock *lock=[NSLock new];
	dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
	for (int i = 0;  i<10000; i ++) {
		dispatch_async(queue, ^{
//			[lock lock];
			[ret addObject:[NSObject new]];
//			[lock unlock];
		});
	}
}


/// ESC_BAD_ACCESS 访问到了已经释放的地址
/// littleName 字符串放在 常量区 不会释放，不会崩溃
///name 放在栈上 会执行release ，多线程会执行多个release导致崩溃。
///name换成其他的对象也是一样的。
- (void)test5{
		dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
		for (int i = 0;  i<100; i ++) {
			dispatch_async(queue, ^{
				NSString *name = [NSString stringWithFormat:@"stringWithFormatstringWithFormat123 %d",i];
				NSLog(@"name: %p",name);
				NSString *littleName = @"123";
				self.person.name= littleName;
			});
		}
}

/// 5. 紧紧当执行到strong的时候监测 weakself是否已经释放，加入没有释放则进行引用，那么久block中执行过程中则不会释放掉，block执行完成，则释放掉
-(void)testBlock5{
	__weak typeof(self) __weakSelf = self;
	[self.block addBlockSecond:^{
		NSLog(@"addBlockSecond test5");

		ViewController2 *strongSelf = __weakSelf;
		NSLog(@"addBlockSecond test5 %@",strongSelf);
		
		
		 strongSelf.name=@"name";
		 NSLog(@"addBlockSecond test5 - 2%@",strongSelf);

		 strongSelf.age = 10;
		 NSLog(@"addBlockSecond test5 - 3%@",strongSelf);

	}];
 }
 
 ///6. 当退出当前界面 该block还未执行的时候，则VC销毁，block不执行。
/// 如果想要执行过程不被中断， 则需要在Block内部strong一次。
/// ViewController2 *strongSelf = __weakSelf即可。
/// 该方式并不影响未执行block之前的对象释放，如果想要一定执行，
/// 则需要
-(void)test6{
	__weak typeof(self) __weakSelf = self;
	[self.block addBlockSecond:^{
//		ViewController2 *strongSelf = __weakSelf;
			NSString *little = @"123";
		NSLog(@"已经开始执行");
		sleep(5);
		__weakSelf.person.name = little;
		NSLog(@"%@ %p",__weakSelf.class,__weakSelf);
	}];
}


///==========================================
一定会执行的写法，没有循环引用。
__weak typeof(self) __weakSelf = self;
	[self.block addBlockSecond:^{
		NSString *name = @"fgyong.cn";
		sleep(5);
		self.person.name = name;
	}];
	
	///7. runloop 与线程 block总和题
	/// 第一个打印 123 原因：队列只有一个线程的时候执行任务在主线程，主线程是默认开启runloop的，所以会打印3
	/// 第二个打印 1 自己创建的线程执行完任务block则生命周期结束，需要保活添加port才能打印3.
	-(void)test7{
	dispatch_queue_t queue=dispatch_queue_create("com.fgyong.cm", DISPATCH_QUEUE_CONCURRENT);
	dispatch_async(queue, ^{
		NSLog(@"1");
		[self performSelector:@selector(pLog) withObject:nil afterDelay:2.0];
		NSLog(@"2");
	});
}
-(void)pLog{
	NSLog(@"3 %d",[NSThread isMainThread]);
}
-(void)test8{
	NSThread *th = [[NSThread alloc]initWithBlock:^{
		NSLog(@"1");
	}];
	[th start];
	//self.th = th;
	[self performSelector:@selector(pLog) withObject:nil afterDelay:2.0];
}


/// 8.  1-99 乱序 100 101，队列是并发队列，任务执行顺序时长不定，导致乱序
/// 栅栏函数可以拦截前100任务，在结束的时候输出100，然后最后执行101
/// 当队列是全局队列则 全部无序，因为栅栏函数无法拦截全局队列
-(void)test9{
//	dispatch_queue_t queue = dispatch_queue_create("com.fgyong.cn", DISPATCH_QUEUE_CONCURRENT);
	dispatch_queue_t queue = dispatch_get_global_queue(0, 0);

	for (int i = 0; i < 100; i ++) {
		dispatch_async(queue, ^{
			NSLog(@"%d",i);
		});
	}
	dispatch_barrier_sync(queue, ^{
		NSLog(@"100");
	});
	dispatch_async(queue, ^{
		NSLog(@"101");
	});
}
  ```
 </details> 