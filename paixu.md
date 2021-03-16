#  排序题：冒泡排序，选择排序，插入排序，快速排序（二路，三路）能写出那些？

这里简单的说下几种快速排序的不同之处，随机快排，是为了解决在近似有序的情况下，时间复杂度会退化为`O(n^2)`,双路快排是为了解决快速排序在大量数据重复的情况下，时间复杂度会退化为`O(n^2)`，三路快排是在大量数据重复的情况下，对双路快排的一种优化。


#### 1. 冒泡排序☀️☀️
时间复杂度O(n^2)

```objc
extension Array where Element : Comparable{
    public mutating func bubbleSort() {
        let count = self.count
        for i in 0..<count {
            for j in 0..<(count - 1 - i) {
                if self[j] > self[j + 1] {
                    (self[j], self[j + 1]) = (self[j + 1], self[j])
                }
            }
        }
    }
}
```

#### 2. 选择排序☀️
时间复杂度O(n^2)

循环每边挑选出来符合条件的用来交换。交换次数比冒泡排序少，性能更好。

```objc
extension Array where Element : Comparable{
    public mutating func selectionSort() {
        let count = self.count
        for i in 0..<count {
            var minIndex = i
            for j in (i+1)..<count {
                if self[j] < self[minIndex] {
                    minIndex = j
                }
            }
            (self[i], self[minIndex]) = (self[minIndex], self[i])
        }
    }
}
```
### 3. 桶排序

时间复杂度O(N)
空间复杂度O(N)

```objc
-(NSArray *)sort:(NSMutableArray*) array{
	NSMutableDictionary<NSNumber *,NSNumber*> * dic = [NSMutableDictionary dictionary];
	/// 记录排序的范围最大和最小值 然后在这个范围内遍历一次即可。
	NSInteger max = NSIntegerMin;
	NSInteger min = NSIntegerMax;
	for (int i = 0; i < array.count; i ++) {
		NSInteger value = [array[i] integerValue];
		if (value > max) {
			max = value;
		}
		if (value < min) {
			min = value;
		}
		NSNumber * number = [NSNumber numberWithInteger:value];
		if ([dic.allKeys containsObject:number]) {
			[dic setObject:@([dic[number] integerValue]+1)  forKey:number];
		}else{
			[dic setObject:@1 forKey:number];
		}
	}
	NSMutableArray *mut=[NSMutableArray arrayWithCapacity:array.count];
	for (NSInteger i = min; i <= max; i ++) {
		NSNumber * number = [NSNumber numberWithInteger:i];
		if ([dic .allKeys containsObject:number]) {
			NSInteger count = [[dic objectForKey:number] integerValue];
			for (NSInteger j = 0; j < count; j++) {
				[mut addObject:number];
			}
		}
	}
	return [mut copy];
}
```
<!--### 3.插入排序☀️

```objc
extension Array where Element : Comparable{
    public mutating func insertionSort() {
        let count = self.count
        guard count > 1 else { return }
        for i in 1..<count {
            var preIndex = i - 1
            let currentValue = self[i]
            while preIndex >= 0 && currentValue < self[preIndex] {
                self[preIndex + 1] = self[preIndex]
                preIndex -= 1
            }
            self[preIndex + 1] = currentValue
        }
    }
}
```-->

### 4.快速排序☀️☀️

1. [参考链接](https://www.jianshu.com/p/bd7d6f1be6e7)
2. [百科](https://baike.baidu.com/item/%E5%BF%AB%E9%80%9F%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/369842?fr=aladdin)

时间复杂度： O(nlogn)

`py` 写法。

```py
/// python 首先从右侧找到比[key]小的值放在[l]位置
/// 然后从左侧找到比[key]大的值放在[right]位置
/// 做完一轮循环 将[key]的值归位 [left]
def quickSort(l:int,r:int,array:list[int]):
    if l >= r :return
    left = l
    right = r
    base = array[l]
    while left < right:
        while left < right and array[right] >= base:
            right -= 1
        array[l] = array[right]
        while left < right and array[left] <= base :
            left += 1
        array[right] = array[left]
    array[left] = base
    quickSort(l,left-1,array)
    quickSort(left+1,r,array)
    
```

`swift` 写法
    
```objc
    
extension Array where Element : Comparable{
    public mutating func quickSort() {
        func quickSort(left:Int, right:Int) {
            guard left < right else { return }
            var i = left + 1,j = left
            let key = self[left]
            while i <= right {
                if self[i] < key {
                    j += 1
                    (self[i], self[j]) = (self[j], self[i])
                }
                i += 1
            }
            (self[left], self[j]) = (self[j], self[left])
            quickSort(left: j + 1, right: right)
            quickSort(left: left, right: j - 1)
        }
        quickSort(left: 0, right: self.count - 1)
    }
}
```

#### 5. 二分查找☀️☀️
**有序**可以用二分查找，无序数组不行，需要遍历，或者先排序在二分。

时间复杂度为O(log2n)

```objc
-(NSInteger)find:(NSArray *)array target:(NSInteger)target{
	NSInteger left = 0,right = array.count -1;
	while (left < right) {
		NSInteger mid = (left+right)/2;
		NSInteger value = [array[mid] integerValue];
		if (value == target) {
			return mid;
		}else if(value < target){
			left=mid;
		}else if(value > target){
			right = mid;
		}
	}
	return  -1;
}
```

#### 6. 随机快排

```objc
extension Array where Element : Comparable{
    public mutating func quickSort1() {
        func quickSort(left:Int, right:Int) {
            guard left < right else { return }
            let randomIndex = Int.random(in: left...right)
            (self[left], self[randomIndex]) = (self[randomIndex], self[left])
            var i = left + 1,j = left
            let key = self[left]
            while i <= right {
                if self[i] < key {
                    j += 1
                    (self[i], self[j]) = (self[j], self[i])
                }
                i += 1
            }
            (self[left], self[j]) = (self[j], self[left])
            quickSort(left: j + 1, right: right)
            quickSort(left: left, right: j - 1)
        }
        quickSort(left: 0, right: self.count - 1)
    }
}
```

#### 7. 双路快排
```objc
extension Array where Element : Comparable{
    public mutating func quickSort2() {
        func quickSort(left:Int, right:Int) {
            guard left < right else { return }
            let randomIndex = Int.random(in: left...right)
            (self[left], self[randomIndex]) = (self[randomIndex], self[left])
            var l = left + 1, r = right
            let key = self[left]
            while true {
                while l <= r && self[l] < key {
                    l += 1
                }
                while l < r && key < self[r]{
                    r -= 1
                }
                if l > r { break }
                (self[l], self[r]) = (self[r], self[l])
                l += 1
                r -= 1
            }
            (self[r], self[left]) = (self[left], self[r])
            quickSort(left: r + 1, right: right)
            quickSort(left: left, right: r - 1)
        }
        quickSort(left: 0, right: self.count - 1)
    }
}
```
##### 8. 三路快排

```objc
// 三路快排

extension Array where Element : Comparable{
    public mutating func quickSort3() {
        func quickSort(left:Int, right:Int) {
            guard left < right else { return }
            let randomIndex = Int.random(in: left...right)
            (self[left], self[randomIndex]) = (self[randomIndex], self[left])
            var lt = left, gt = right
            var i = left + 1
            let key = self[left]
            while i <= gt {
                if self[i] == key {
                    i += 1
                }else if self[i] < key{
                    (self[i], self[lt + 1]) = (self[lt + 1], self[i])
                    lt += 1
                    i += 1
                }else {
                    (self[i], self[gt]) = (self[gt], self[i])
                    gt -= 1
                }
                
            }
            (self[left], self[lt]) = (self[lt], self[left])
            quickSort(left: gt + 1, right: right)
            quickSort(left: left, right: lt - 1)
        }
        quickSort(left: 0, right: self.count - 1)
    }
}
```