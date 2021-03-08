### 1. `setNeedDisplay`和`layoutifNeeded`什么关系？

`UIView`的`setNeedsDisplay`和`setNeedsLayout`两个方法都是异步执行的，而`setNeedsDisplay`会自动调用`drawRect`方法，这样可以拿到`UICraphicsGetCurrent`进行绘制，而`setNeedsDisplay`会默认调用`layoutSubviews`，给当前的视图做标记，`layoutifNeeded`查找是否标记，如果有标记会立即刷新。只有`setNeedslayout`和`layoutIfNeeded`二者结合起来使用，才能立即刷新。