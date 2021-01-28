

### 1. 线程之间通讯
<details>
  <summary>点击查看详细内容</summary>
   线程之间通过NSPort通讯，将NSPort加入到runloop中，可以通过设置port的代理来监听该端口接受的数据，其实是先讲数据发给runloop，然后经过runloop中转到delegate去处理数据。
   
[查看具体代码]((https://github.com/ifgyong/demo/tree/master/OC/%E7%BA%BF%E7%A8%8B%E9%80%9A%E8%AE%AF))  
</details>  
