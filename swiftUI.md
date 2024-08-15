## SwiftUI
### 布局
#### ZStack
层叠布局或者叫做绝对布局
| 对齐方式 | 示例|
| --- | --- |
|`top`|![image](https://github.com/user-attachments/assets/f50e5f16-768f-4285-8de2-b9a475c8e06a)|
|topLeading|![image](https://github.com/user-attachments/assets/20be88a0-9b67-4158-8d8d-c08e947c00cf)|
|topTrailing|![image](https://github.com/user-attachments/assets/987871d0-1800-494e-bcb9-45a809fa0bb7)|
|bottom|![image](https://github.com/user-attachments/assets/9a66f86a-aea1-45f5-b69c-e5b86d2ef4d3)|
|bottomLeading|![image](https://github.com/user-attachments/assets/2c13a4e3-2e22-40d9-941c-ce688c53265b)|
|bottomTrailing|![image](https://github.com/user-attachments/assets/05f71c8b-3799-4523-99e4-bd4f0746230b)|

```
struct ContentView: View {
    @State var textFieldValue:String;
         let colors: [Color] =
             [.red, .orange, .yellow, .green, .blue, .purple]
    
    var body: some View {
        VStack(content: {
            ZStack(content: {
                // 设置背景色
                Rectangle().fill(Color.gray.opacity(0.2))
                    .frame(height: 80) // 根据需要调整高度
                Text("我是Label：")
                    .multilineTextAlignment(.leading)
                    .font(.system(size: 20))
                    .foregroundColor(Color.blue)
                    .border(Color.red, width: 1)
            })
            ZStack(alignment: .topLeading) {
                Rectangle()
                    .fill(Color.red)
                    .frame(width: 100, height: 50)
                Rectangle()
                    .fill(Color.blue)
                    .frame(width:50, height: 150)
            }
            .border(Color.green, width: 1)
        })
    }
    
}
```
#### HStack


水平布局，元素按照水平方式顺序排列
![image](https://github.com/user-attachments/assets/34e2bca5-2a6e-4aa4-941c-cf5ea935d06c)



```
    HStack{
            ZStack(content: {
                // 设置背景色
                Rectangle().fill(Color.red.opacity(0.2))
                    .frame(height: 80) // 根据需要调整高度
                Text("左边")
            })
            ZStack(content: {
                // 设置背景色
                Rectangle().fill(Color.gray.opacity(0.2))
                    .frame(height: 80) // 根据需要调整高度
            })
            ZStack(content: {
                // 设置背景色
                Rectangle().fill(Color.red.opacity(0.2))
                    .frame(height: 80) // 根据需要调整高度
                Text("右边")
            })
        }
```

#### VStack
垂直布局，元素按照垂直方式顺序排列

![image](https://github.com/user-attachments/assets/f003d35d-04b3-4e04-91bc-236d156aea84)

```
  var body: some View {
        VStack(content: {
            ZStack(content: {
                // 设置背景色
                Rectangle().fill(Color.red.opacity(0.2))
                    .frame(height: 80) // 根据需要调整高度
                Text("上边")
            })
            
            ZStack(content: {
                // 设置背景色
                Rectangle().fill(Color.gray.opacity(0.2))
                    .frame(height: 80) // 根据需要调整高度
            })
            ZStack(content: {
                // 设置背景色
                Rectangle().fill(Color.red.opacity(0.2))
                    .frame(height: 80) // 根据需要调整高度
                Text("下边")
            })
        })
    }
```

#### ScrollView

|滚动方向|示例|
|---|---|
|horizontal|![2024-08-15 20-17-16 2024-08-15 20_17_26](https://github.com/user-attachments/assets/4a3bc85d-8f2d-490c-923d-16e1f681eb6a)|
|vertical|![3967DE98-8973-4FFF-8694-5A2E6BC32EB0-89637-00004EEE5B4718D5](https://github.com/user-attachments/assets/271da686-c7ea-4778-b450-e62687267ab3)|

```
  // 水平滚动视图
        ScrollView(.horizontal) {
            HStack {
                ForEach(0..<5) { index in
                    ZStack{
                        Rectangle().fill(colors[index])
                            .frame(width: 200, height: 80)
                        Text("Item \(index)")
                            .padding()
                    }
                }
            }
     // 垂直滚动视图
        ScrollView(.vertical) {
            VStack {
                ForEach(0..<5) { index in
                    ZStack{
                        Rectangle().fill(colors[index])
                            .frame(width: 200, height: 80)
                        Text("Item \(index)")
                            .padding()
                    }
                }
            }
        }
```


