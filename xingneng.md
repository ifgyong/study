# 性能优化
### 1. 对象放在子线程销毁对象
 <details>
  <summary>点击查看详细内容</summary>

```  	
self.person=[Person new];
Person *p2 = self.person;
self.person=nil;
dispatch_async(queue, ^{
	[p2 class];
});
```
</details>

### 2. 子线程绘画位图然后显示

