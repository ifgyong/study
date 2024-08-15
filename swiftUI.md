## SwiftUI
### 布局
#### ZStack
层叠布局或者叫做绝对布局
| 对齐方式 | 示例|
| --- | --- |
|`top`|![image](https://github.com/user-attachments/assets/8c066178-3d95-4078-8dc6-d25b472bea69)|
|topLeading|![image](https://github.com/user-attachments/assets/49be1e5b-5bf3-4fb0-9527-32bd1f967570)|
|topTrailing|![image](https://github.com/user-attachments/assets/9f3755aa-5d58-49d7-af7c-a0e5fa7dd278)|
|bottom|![image](https://github.com/user-attachments/assets/f3f07f32-5c66-476f-be8a-6484abf1eda8)|
|bottomLeading|![image](https://github.com/user-attachments/assets/a7414481-ff34-447f-9490-df00f54ef39e)|
|bottomTrailing|![image](https://github.com/user-attachments/assets/15b1fa69-7ead-4ad3-84c2-20109835ffd9)|

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
![image](https://github.com/user-attachments/assets/57ce119f-d948-4922-9e32-568018593cb8)


```
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
```

#### VStack
垂直布局，元素按照垂直方式顺序排列


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


