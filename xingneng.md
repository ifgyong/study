### 1. 放在子线程销毁对象
 <details>
  <summary>点击查看详细内容</font></summary>
  <pre><code>
  	self.person=[Person new];
	Person *p2 = self.person;
	self.person=nil;
	dispatch_async(queue, ^{
		[p2 class];
	});
  </code>
</details>