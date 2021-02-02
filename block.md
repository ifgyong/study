# Block


## [1. Block 直接看题](./block_ti.md)、[Block知识点图](./img/block.png)
## Block 分类
#### **一共6中，常用的有三种。**

```
void * _NSConcreteStackBlock[32] = { 0 };
void * _NSConcreteMallocBlock[32] = { 0 };
void * _NSConcreteAutoBlock[32] = { 0 };
void * _NSConcreteFinalizingBlock[32] = { 0 };
void * _NSConcreteGlobalBlock[32] = { 0 };
void * _NSConcreteWeakBlockVariable[32] = { 0 };
```

#### 分类
#### 1. __NSGlobalBlock__
 
 **未捕获外部变量，默认是`__NSGlobalBlock__`类型。**
 
```
dispatch_block_t block =^{};
dispatch_block_t block_1 =[^{} copy];
```
#### 2. __NSStackBlock__
 
 在`mrc`下捕获外部变量，则是栈`block`，在`arc`情况下，系统自动执行`copy`，`block`类型变为`__NSMallocBlock__`,如果是`static` 或者**全局变量**则不需要捕获。因为全局变量在全局都可以访问到。
 
```
int a = 0;
dispatch_block_t block2 =^{
	NSLog(@"%d",a);
};
```
#### 3. __NSMallocBlock__

在`arc`编译环境下，系统自动管理`block_copy`将`stackBlock`类型变为`__NSMallocBlock__`，将`block copy`之后赋值给强引用的指针。

```
int a = 0;
dispatch_block_t block2 =^{
	NSLog(@"%d",a);
};
```
## 2. Block

```
clang -x objective-c -rewrite-objc -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator.sdk main.m
```

### 捕获外接变量

生成的结构体block内部直接生成了包含外部变量，在构造函数中复制给结构体的变量a。



```


int main(int argc, const char * argv[]) {
	int a = 10;
	void (^block)(void)=^{
		printf("fgyong.cn %d",a);
	};
	block();
	return 0;
}

============================clang 之后的代码=========================

struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  int a;//属性
	// 构造函数 自动复制给a=_a
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int _a, int flags=0) : a(_a) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
int main(int argc, const char * argv[]) {
 int a = 10;
 // 步骤A 构造结构体
 void (*block)(void)=(__main_block_impl_0(__main_block_func_0, &__main_block_desc_0_DATA, a));
 // 步骤B 执行block 看下面C
 block->FuncPtr((__block_impl *)block);
 return 0;
}

//步骤C 当执行则取出来结构体中存储的a值。
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
  int a = __cself->a; // 值拷贝 只能取出来不能赋值。

  printf("fgyong.cn %d",a);
 }
```

#### __block 修饰变量
**__block修饰的变量的地址被block结构体保存，地址拷贝，可以对对象进行赋值。**

```
struct __Block_byref_a_0 {
  void *__isa;
__Block_byref_a_0 *__forwarding;//指向自己的地址
 int __flags;
 int __size;//占用空间大小
 int a;//变量的值
};

struct __main_block_impl_0 {
  
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  __Block_byref_a_0 *a; // 记录外部变量的结构体的地址，该结构体记录了变量的值与地址
  
	// 构造函数将 __block修饰的a的地址保存到了block的结构体中。
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, __Block_byref_a_0 *_a, int flags=0) : a(_a->__forwarding) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
// 执行block
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
  __Block_byref_a_0 *a = __cself->a;//指针拷贝 可以赋值

  printf("fgyong.cn %d",(a->__forwarding->a));
 }
static void __main_block_copy_0(struct __main_block_impl_0*dst, struct __main_block_impl_0*src) {_Block_object_assign((void*)&dst->a, (void*)src->a, 8/*BLOCK_FIELD_IS_BYREF*/);}

static void __main_block_dispose_0(struct __main_block_impl_0*src) {_Block_object_dispose((void*)src->a, 8/*BLOCK_FIELD_IS_BYREF*/);}

static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
  void (*copy)(struct __main_block_impl_0*, struct __main_block_impl_0*);
  void (*dispose)(struct __main_block_impl_0*);
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0), __main_block_copy_0, __main_block_dispose_0};
int main(int argc, const char * argv[]) {
 __attribute__((__blocks__(byref))) __Block_byref_a_0 a =
	{(void*)0,
	 (__Block_byref_a_0 *)&a,
	 0,
	 sizeof(__Block_byref_a_0),
	 10};
 void (*block)(void)=(__main_block_impl_0(__main_block_func_0, __main_block_desc_0_DATA,&a, 570425344));
 block->FuncPtr((__block_impl *)block);
 return 0;
}
```

