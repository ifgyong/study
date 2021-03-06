# Block


## 1. Block 线程 runloop综合题
 <!--<details>
  <summary>点击查看详细内容</summary>-->
  
  ```objc
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

/// 5. 紧紧当执行到strong的时候监测 weakself是否已经释放，
/// 假如没有释放则进行引用，那么久block中执行过程中则不会释放掉
/// block执行完成，则释放掉
-(void)testBlock5{
	__weak typeof(self) __weakSelf = self;
	[self.block addBlockSecond:^{
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
//<1> 这种必须执行block才行，否则block中的strongSelf没有执行，则self会变为nil。
-(void)test6{
	__weak typeof(self) __weakSelf = self;
	[self.block addBlockSecond:^{
		ViewController2 *strongSelf = __weakSelf;
			NSString *little = @"123";
		NSLog(@"已经开始执行");
		
		dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(4 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
				sleep(5);
		__weakSelf.person.name = little;
		NSLog(@"%@ %p",__weakSelf.class,__weakSelf);
		});
	}];
}


///==========================================
	<2> 或者直接前引用，到任务执行完毕重置为nil
	直接vc被gcd强引用，直到被释放才打破环，可以被释放。
==================================================
-(void)t1{
	self.name = @"fgyong.cn";
	__block ViewController2 *vc= self;
	self.block = ^{
		dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(4 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
				NSLog(@"hello word %@",vc.name);
			vc=nil;
		});
	};
	self.block();
}
-(void)dealloc{
	NSLog(@"%s",__func__);
}
====================================================
<3> 通过传参将数据直接传过去，不通过引用。

typedef   void (^FYBlock)(id data);

	self.name = @"fgyong.cn";
	self.block = ^(id data) {
		dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(4 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
			NSLog(@"hello word %@",data);
		});
	};
	self.block(self.name);
	
	
==================================================
	
	///7. runloop 与线程 block总和题
	/// 第一个打印 123 原因：队列只有一个线程的时候执行任务在主线程，主线程是默认开启runloop的，所以会打印3
	/// 第二个打印 1 自己创建的线程执行完任务block则生命周期结束
	/// 需要保活添加port才能打印3
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
	// [self performSelector:@selector(pLog) withObject:nil afterDelay:2.0];
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

// 10 同步队列执行顺序
/// 12是异步任务，3 是同步任务，
/// 0是同步主线程任务，所以3在0前边
/// 7 8 9 无序 1 2 3无序，但是3在0前边。
/// 最终是[1,2,3],0,[7,8,9]
/// 只有0和3是有确定位置的，其他的两个片段都是无序的。
	dispatch_queue_t queu=dispatch_queue_create("com.fgyong.cn", DISPATCH_QUEUE_SERIAL);
	dispatch_async(queu, ^{
		NSLog(@"1");
	});
	dispatch_async(queu, ^{
		NSLog(@"2");
	});
	dispatch_sync(queu, ^{
		NSLog(@"3 %@",[NSThread currentThread]);
	});
	NSLog(@"0");
	dispatch_async(queu, ^{
		NSLog(@"7");
	});
	dispatch_async(queu, ^{
		NSLog(@"8");
	});
	dispatch_async(queu, ^{
		NSLog(@"9");
	});
	
	
	/// 11. 最终a输出多少
	/// 当任务基本一致则基本遵循FIFO原则
	/// 只需要在队尾添加打印任务即可，就是真实的a的值
	__block int a = 0;
	while (a<10) {
		dispatch_async(dispatch_get_global_queue(0, 0), ^{
			a +=1;
			NSLog(@"%d",a);

		});
	}
	NSLog(@"ret==%d",a);
//	dispatch_async(dispatch_get_global_queue(0, 0), ^{
//		NSLog(@"ret==%d",a);
//	});


  ```
 </details> 