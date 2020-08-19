# 第一节课  算法概念  插入排序（Insert Sort） 合并排序 (Merge Sort)

# 一 写程序要注意的重要事项（important）

正确(correctness)、简洁(simplicity)、健壮(robustness)、安全(security)、

模块化(modularity)、用户友好(user-friendly)、易于维护(maintainability)、

成本小(cost)、

高效(efficient)



严蔚敏《数据结构》算法特征和设计要求

**特征：有穷性、确定性、可行性、输入、输出**

要求：正确性(correctness)、可读性(readability)、健壮性(robustness 鲁棒性)、效率与低存储量要求



# 二 插入排序(Insertion Sort)

**直接插入排序（Insertion Sort）**简单直观，基本思想是将待排记录按关键字插入到前面已经排好的序列中，对于未排数据，从已经排列的数据从后往前扫描，找到相应的位置插入。所以在从后往前扫描时，需要反复将已经排序的元素逐步后挪。

###     2.1算法描述

一般来说，插入排序都采用in-place在数组上实现。具体算法描述如下：

- 步骤1: 从第一个元素开始，该元素可以认为已经被排序；
- 步骤2: 取出下一个元素，在已经排序的元素序列中从后向前扫描。如果该元素（已排序）大于新元素，将该元素移到下一位置，直到找到已排序的元素小于或者等于新元素的位置，将新元素插入到该位置后；
- 步骤3: 重复步骤2。



###      2.2算法演示

![](E:/Desktop/GitHub/Algorithm/sort/排序.assets/charu.gif)



### 2.3代码实现

```c++
void insertion_sort(int arr[],int len){
        for(int i=1;i<len;i++){
                int key=arr[i];
                int j=i-1;
                while((j>=0) && (key<arr[j])){
                        arr[j+1]=arr[j];
                        j--;
                }
                arr[j+1]=key;
        }
}
```

###    2.4算法分析

#### 复杂度

 		**空间上**：只需一个记录的辅助空间

​         **时间上**：排序的基本操作为比较和移动

​        		 **最佳情况** 当待排列的记录按关键字非递减有序排列（正序）时，所需进行关键字之间的比较次数最小为 **n-1**  ($\sum_{i=2}^n1$), 记录不需要移动, $O(n)$

​			 	**最差情况** 当当待排列的记录按关键字非递增有序排列（逆序）时，所需进行关键字之间的比较次数最大**(n+2)(n-1)/2**        ($\sum_{i=2}^ni$ )  , 记录移动次数也变为最大 **(n+4)(n-1)/2**      ( $\sum_{i=2}^n{i+1}$)

​    			**平均情况** 可取上述最大值和最小值的平均值，$O(n^2)$

#### 稳定性

​          由于只需要找到不大于当前数的位置而并不需要交换，因此，直接插入排序是稳定的排序方法

#### 使用场景

​    插入排序由于 $O(n^2)$的复杂度，在数组较大的时候不适用。但是，在数据比较少的时候，是一个不错的选择，一般做为快速排序的扩充。例如，在**STL**的sort算法和**stdlib**的**qsort**算法中，都将插入排序作为快速排序的补充，用于少量元素的排序。又如，在**JDK 7 java.util.Arrays**所用的**sort**方法的实现中，当待排数组长度小于47时，会使用插入排序。

# 三 合并排序(Merge Sort)

### 3.1 算法描述

归并排序算法完全遵循分治模式。 直观上其操作如下：

**分解**： 分解待排序的$n/2$ 个元索的序列成各具$n/2$个元索的两个子序列。

**解决**： 使用归并排序递归地排序两个子序列。

**合并**： 合并两个已排序的子序列以产生已排序的答案  

### 3.2算法演示



![](第一节课  算法概念  插入排序（Insert Sort） 合并排序 (Merge Sort).assets/merging Sort.gif)

### 3.3算法实现

​	C++

**递归版**：

```c++
void MergeSort(vector<int> &Array, int front, int end) {
    if (front >= end)
        return;
    int mid = (front + end) / 2; //将Array平分
    MergeSort(Array, front, mid);//递归地将Array[front,mid]归并为有序
    MergeSort(Array, mid + 1, end);//递归地将Array[mid+1,end]归并为有序
    Merge(Array, front, mid, end);//将Array[front,mid]，Array[mid+1,end]合并
}
void Merge(vector<int> &Array, int front, int mid, int end) {
    // preconditions:
    // Array[front...mid] is sorted
    // Array[mid+1 ... end] is sorted
    // Copy Array[front ... mid] to LeftSubArray
    // Copy Array[mid+1 ... end] to RightSubArray
    vector<int> LeftSubArray(Array.begin() + front, Array.begin() + mid + 1);
    vector<int> RightSubArray(Array.begin() + mid + 1, Array.begin() + end + 1);
    int idxLeft = 0, idxRight = 0;
    LeftSubArray.insert(LeftSubArray.end(), numeric_limits<int>::max());
    RightSubArray.insert(RightSubArray.end(), numeric_limits<int>::max());
    // Pick min of LeftSubArray[idxLeft] and RightSubArray[idxRight], and put into Array[i]
    for (int i = front; i <= end; i++) {
        if (LeftSubArray[idxLeft] < RightSubArray[idxRight]) {
            Array[i] = LeftSubArray[idxLeft];
            idxLeft++;
        } else {
            Array[i] = RightSubArray[idxRight];
            idxRight++;
        }
    }
}
```

###  3.4 算法分析

我们分析建立归并排序$n$个数的最坏情况运行时间$T(n)$的递归式。

**分解**： 分解仅仅计算子数组的中间位置，需要常量时间 $\Theta(1)$

**解决**：递归地求解两个规模均为$n/2$的子问题，将贡献$2T(n/2)$的运行时间

**合并**： 在一个具有$n$个元素的子数组上进行MERGE需要$\Theta(n)$的时间


$$
T(n)= \{ \begin{aligned}
		&c   &若n=1 \\
		&2T(n/2)+\Theta(n)	&若n>1
\end{aligned} \\
重写为 \ \ 

T(n)= \{ \begin{aligned}
		&c   &若n=1 \\
		&2T(n/2)+cn	&若n>1
\end{aligned}
$$
**递归树直观理解**

<img src="第一节课  算法概念  插入排序（Insert Sort） 合并排序 (Merge Sort).assets/1.png" style="zoom:200%;" />


$$
cnlgn+cn\in \Theta(nlgn)
$$

