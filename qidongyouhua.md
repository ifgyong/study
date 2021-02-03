# 启动

## 1. 启动顺序
<details>
  <summary>点击查看详细内容</summary>
  - 1. `framework initializers`
  - 2. image `+load`
  - 3. `c/c++ __attribute__`
  - 4. 所有 `initalizers`

  > `+load` 函数父类先于子类，类先于类别。
  
  
![](media/16123179478932.jpg)

</details>