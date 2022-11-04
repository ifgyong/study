### 1. `setNeedDisplay`和`layoutifNeeded`什么关系？

`UIView`的`setNeedsDisplay`和`setNeedsLayout`两个方法都是异步执行的，而`setNeedsDisplay`会自动调用`drawRect`方法，这样可以拿到`UICraphicsGetCurrent`进行绘制，而`setNeedsDisplay`会默认调用`layoutSubviews`，给当前的视图做标记，`layoutifNeeded`查找是否标记，如果有标记会立即刷新。只有`setNeedslayout`和`layoutIfNeeded`二者结合起来使用，才能立即刷新。

### 2. btn扩大相应范围

1. 自定义`View`自己实现`hitTest:(CGPoint)point withEvent:(UIEvent *)event`方法，再函数体内部实现判断是否拦截手势。

```
#import "TestView.h"

@implementation TestView

- (instancetype)initWithFrame:(CGRect)frame
{
    self = [super initWithFrame:frame];
    if (self) {
        [self loadUI];
    }
    return self;
}

- (void)loadUI {
    self.button = [UIButton buttonWithType:UIButtonTypeCustom];
    self.button.frame = CGRectMake(50, 50, 100, 100);
    self.button.backgroundColor = UIColor.purpleColor;
    [self.button addTarget:self action:@selector(buttonClick) forControlEvents:UIControlEventTouchUpInside];
    [self addSubview:self.button];
}

// 此处拦截事件传递
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event {
    if (CGRectContainsPoint(self.bounds, point)) {
        return self.button;
    }
    return [super hitTest:point withEvent:event];
}

- (void)buttonClick {
    NSLog(@"点击");
}
```

2. 继承`UIButton`,重写`-(BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event`该方法返回一个布尔值,
 `YES`说明点击事件在按钮`frame`中,`NO`说明点击事件不在按钮的`frame`中,因此将按钮`frame`周围等距离的区域内的点击事件都返回`YES`,便达到了扩大响应范围的目的。

```
TestButton.h文件
#import <UIKit/UIKit.h>

NS_ASSUME_NONNULL_BEGIN
@interface TestButton : UIButton
@property (nonatomic, assign) CGSize touchSize;
@end
NS_ASSUME_NONNULL_END

TestButton.m文件
#import "TestButton.h"

@implementation TestButton

- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event {
    
    CGRect bounds = self.bounds;
    CGFloat widthDelta = MAX(self.touchSize.width - bounds.size.width, 0);
    CGFloat heightDelta = MAX(self.touchSize.height - bounds.size.height, 0);
    CGRect NewBounds = CGRectInset(bounds, - 0.5 * widthDelta, - 0.5 * heightDelta);
    return CGRectContainsPoint(NewBounds, point);
}

@end


调用
self.button = [TestButton buttonWithType:UIButtonTypeCustom];
self.button.frame = CGRectMake(50, 100, 200, 50);
self.button.backgroundColor = UIColor.orangeColor;
[self.button addTarget:self action:@selector(click) forControlEvents:UIControlEventTouchUpInside];
[self.view addSubview:self.button];
self.button.touchSize = CGSizeMake(300, 100);
```
