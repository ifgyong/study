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
- 1. 图片**下采样**
- 2. 尽量少离屏渲染
- 3. 常用数据一定缓存`内存缓存`->`磁盘缓存`->`网络加载`
- 4. 异步渲染，将渲染好的`img`赋值给`content`。
- 5. 预加载数据，预渲染，减少卡顿时间。
- 6. 

