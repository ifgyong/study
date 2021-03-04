# Masonry
[参考1](https://draveness.me/ios-yuan-dai-ma-fen-xi-masonry/)

## 总结
### `Masonry` 如何为视图添加约束(面试回答)?
`Masonry` 与其它的第三方开源框架一样选择了使用分类的方式为 `UIKit` 添加一个方法 `mas_makeConstraint`, 这个方法接受了一个 `block`, 这个 `block` 有一个 `MASConstraintMaker` 类型的参数, 这个 `maker` 会持有一个约束的数组, 这里保存着所有将被加入到视图中的约束.

我们通过链式的语法配置 `maker`, 设置它的 `left right` 等属性, 比如说 `make.left.equalTo(view)`, 其实这个 `left equalTo` 还有像 `with offset `之类的方法都会返回一个 `MASConstraint` 的实例, 所以在这里才可以用类似 `Ruby `中链式的语法.

在配置结束后, 首先会调用` maker` 的 `install` 方法, 而这个` maker` 的` install` 方法会遍历其持有的约束数组, 对其中的每一个约束发送 `install` 消息. 在这里就会使用到在上一步中配置的属性, 初始化 `NSLayoutConstraint` 的子类 `MASLayoutConstraint` 并添加到合适的视图上.

视图的选择会通过调用一个方法 `mas_closestCommonSuperview:` 来返回两个视图的最近公共父视图.