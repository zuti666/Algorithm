# STL swap()

在进行算法学习时，使用到了***sort()***函数以及***swap()***函数 ,由于对c++的STL不是很熟悉，特意记录先介绍***swap()***函数

| 函数名     | 头文件      | 函数功能                                                     |
| ---------- | ----------- | ------------------------------------------------------------ |
| swap       | <algorithm> | 交换存储在两个对象中的值                                     |
|            | 函数原型    | template<typename _Tp, size_t _Nm>  void  swap(_Tp (&__a)[_Nm], _Tp (&__b)[_Nm]) |
| swap_range | <algorithm> | 将指定范围内的元素与另一序列元素值进行交换                   |
|            | 函数原型    | template<typename _FIter1, typename _FIter2>  _FIter2   swap_ranges(_FIter1, _FIter1, _FIter2); |

```c++
template<typename _Tp, size_t _Nm>   
void    
swap(_Tp (&__a)[_Nm], _Tp (&__b)[_Nm])
#if __cplusplus >= 201103L    
 noexcept(noexcept(swap(*__a, *__b)))
#endif;  
template<typename _FIter1, typename _FIter2>    
_FIter2    
swap_ranges(_FIter1, _FIter1, _FIter2);
```



**链接 https://zhuanlan.zhihu.com/p/36274119。****介绍了sort()函数。下面是链接内容。**

**还有两个对sort函数的分析
**

**http://feihu.me/blog/2014/sgi-std-sort/**

**https://www.cnblogs.com/likeghee/p/11565503.html**

**
**

## ***std::sort适合哪些容器***

这么高效的算法，是不是所有的容器都可以使用呢？我们常规数组是否也能使用？我们知道在STL中的容器可以大致分为：

- 序列式容器：vector, list, deque
- 关联式容器：set, map, multiset, multimap
- 配置器容器：queue, stack, priority_queue
- 无序关联式容器：unordered_set, unordered_map, unordered_multiset, unordered_multimap。这些是在C++ 11中引入的

对于所有的关联式容器如map和set，由于它们底层是用红黑树实现，因此已经具有了自动排序功能，不需要`std::sort`。至于配置器容器，因为它们对出口和入口做了限制，比如说先进先出，先进后出，因此它们也禁止使用排序功能。

由于`std::sort`算法内部需要去取中间位置元素的值，为了能够让访问元素更迅速，因此它只接受有随机访问迭代器的容器。对于所有的无序关联式容器而言，它们只有前向迭代器，因而无法调用`std::sort`。但我认为更为重要的是，从它们名称来看，本身就是无序的，它们底层是用哈希表来实现。它们的作用像是字典，为的是根据key快速访问对应的元素，所以对其排序是没有意义的。

剩下的三种序列式容器中，vector和deque拥有随机访问迭代器，因此它们可以使用`std::sort`排序。而list只有双向迭代器，所以它无法使用`std::sort`，但好在它提供了自己的sort成员函数。

另外，我们最常使用的数组其实和`vector`一样，它的指针本质上就是一种迭代器，而且是随机访问迭代器，因此也可以使用`std::sort`

## 实现逻辑

STL的sort算法，数据量大时采用QuickSort快排算法，分段归并排序。一旦分段后的数据量小于某个门槛（16），为避免QuickSort快排的递归调用带来过大的额外负荷，就改用Insertion Sort插入排序。如果递归层次过深，还会改用HeapSort堆排序。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/vkC2hqLZ6XHjAV7ywUnnbQCgFIR863C2vWpm2AxJxCNK3e8rokEUkLPMyNIRGqWls7jAMyIdy6uSF6U8C0sbiaw/640?wx_fmt=jpeg)

结合快速排序-插入排序-堆排序 三种排序算法。

## 具体代码

源文件：

```c++
C:\Program Files (x86)\Dev-Cpp\MinGW64\lib\gcc\x86_64-w64-mingw32\4.9.2\include\c++\bits

template<typename _RandomAccessIterator, typename _Compare>   
inline void    
sort(_RandomAccessIterator __first, _RandomAccessIterator __last,   _Compare __comp)
{      
    // concept requirements      
    //迭代器类型得是Mutable RandomAccessIterator      
    //迭代器所指类型得是支持<运算符的       
 	__glibcxx_function_requires(_Mutable_RandomAccessIteratorConcept<      			      _RandomAccessIterator>)      			               __glibcxx_function_requires(_BinaryPredicateConcept<_Compare,      
	typename iterator_traits<_RandomAccessIterator>::value_type,      
    typename iterator_traits<_RandomAccessIterator>::value_type>)     
    //检查范围      
	__glibcxx_requires_valid_range(__first, __last);      
    std::__sort(__first, __last,__gnu_cxx::__ops::__iter_comp_iter(__comp));    
}
```

第一层已经读完了，做的内容主要是调用第二层的__sort函数

***__sort***

```c++
 // sort  
template<typename _RandomAccessIterator, typename _Compare>    
inline void    
__sort(_RandomAccessIterator __first, _RandomAccessIterator __last,     _Compare __comp)    
{      
    if (__first != __last)  {    
        //进行快速排序，执行完后基本有序，O（nlogn）   
        //__introsort_loop会在一开始会判断区间的大小，当区间小于16的时候，就直接返回     
        std::__introsort_loop(__first, __last,        std::__lg(__last - __first) * 2,        __comp);     
        // 在区间基本有序的基础上再做一遍插入排序，使区间完全有序   
        std::__final_insertion_sort(__first, __last, __comp);  
    }   
}
```

 

其中`__lg`函数是计算递归深度，即计算log(n)用来控制分割恶化，当递归深度达到该值改用堆排序，因为堆排序是时间复杂度恒定为nlogn：

```c++
template<typename _Size>    
inline _Size    
__lg(_Size __n)    
{     
_Size __k;      
for (__k = 0; __n != 1; __n >>= 1)//n>>=1 按位于，即除以2        
	++__k;      
return __k;    
}
```

***__introsort_loop，快速排序/堆排序\***

先来看，`__introsort_loop` 快排实现部分：对于区间小于`16`的采用快速排序，如果递归深度恶化改用`堆排序`。当数据量很大时，Introspective Sort都是采用快速排序，也就是__unguarded_partition，当last-first<16时，再使用递归来排序显然不划算，递归的开销相对来说太大，这时候将退出函数执行__final_insertion_sort插入排序。

该函数__introsort_loop退出条件是last-first < __stl_threshold即区域小于等于阈值16，或者超过递归深度阈值depth_limit，函数退出后进入到__final_insertion_sort插入排序，（这个时候序列的每个子序列都有相当程度的排序，但又尚未完全排序，使用插入排序再好不过）

另外，在递归部分，快速排序也做了优化，使用了单边循环的方式做递归。在用__unguarded_partition快排后取到index做cut，递归进入__introsort_loop，区间是[cut, last)，下一行又将last赋值成cut即last = cut；单边递归完成后回到开始，又开始__unguarded_partition快排，此时的边区间就到了[first，cut）,构成了一个左右的循环结构，而不是平时写的快排左右的递归结构，STL这样写节省了一半的函数递归调用，大大减少了开销

还有一点优化是pivot的取值，__median函数，它取首部、尾部和中部三个元素的中值作为pivot，而我们平时直接用首部，中部或者尾部来做pivot，通常我们并不比较他们的取值，在很多情况下这将引起递归的恶化，depth_limit就是用来判断分割行为是否有恶化倾向的阈值，当depth_limit减到0时将采用partial_sort堆排序。



```c++
/// This is a helper function for the sort routine.  
template<typename _RandomAccessIterator, typename _Size, typename _Compare>    
void   
__introsort_loop(_RandomAccessIterator __first,         _RandomAccessIterator __last,         _Size __depth_limit, _Compare __comp)    
{     
	//_S_threshold=16，每个区间必须大于16才递归   
	//while语句判断，last-first > __stl_threshold，__stl_threshold为最小分段阈值，值为16，   	//大体上来说，当数据量很大时，Introspective Sort都是采用快速排序，也就是__unguarded_partition，当last-first<16时，再使用递归来排序显然不划算，递归的开销相对来说太大，这时候将退出函数执行__final_insertion_sort插入排序。      
	while (__last - __first > int(_S_threshold))  
	{  //当快速排序的层数过大时，改用堆排序    
	if (__depth_limit == 0)      
		{        
		std::__partial_sort(__first, __last, __last, __comp);        
		return;      
		}    
	--__depth_limit;   
	//将数据分为两个集合    
	_RandomAccessIterator __cut =      std::__unguarded_partition_pivot(__first, __last, __comp);    
	//将后一半递归    
	std::__introsort_loop(__cut, __last, __depth_limit, __comp);    
	//继续排序前一半    
	__last = __cut;  } 
	//单边循环的方式做递归而不是平时写的快排左右的递归结构， //STL这样写节省了一半的函数递归调用，大大减少了开销    
}
```

### ***递归结构***

可以看出它是一个递归函数，因为我们说过，Introspective Sort在数据量很大的时候采用的是正常的快速排序，因此除了处理恶化情况以外，它的结构应该和快速排序一致。但仔细看以上代码，先不管循环条件和`if`语句(它们便是处理恶化情况所用)，循环的后半部分是用来递归调用快速排序。但它与我们平常写的快速排序有一些不同，对比来看，以下是我们平常所写的快速排序的伪代码：

```c++
function quicksort(array, left, right)   
// If the list has 2 or more items    
if left < right        
    // See "#Choice of pivot" section below for possible choices        
    choose any pivotIndex such that left ≤ pivotIndex ≤ right        
    // Get lists of bigger and smaller items and final position of pivot        
    pivotNewIndex := partition(array, left, right, pivotIndex)        
    // Recursively sort elements smaller than the pivot (assume pivotNewIndex - 1 does not underflow)        
    quicksort(array, left, pivotNewIndex - 1)        
    // Recursively sort elements at least as big as the pivot (assume pivotNewIndex + 1 does not overflow)        
    quicksort(array, pivotNewIndex + 1, right)
```

`__introsort_loop`中只有对右边子序列进行递归调用是不是？左边的递归不见了。的确，这里的写法可读性相对来说比较差，但是仔细一分析发现是有它的道理的，它并不是没有管左子序列。注意看，在分割原始区域之后，对右子序列进行了递归，接下来的`last = cut`将终点位置调整到了分割点，那么此时的`[first, last)`区间就是左子序列了。又因为这是一个循环结构，那么在下一次的循环中，左子序列便得到了处理。只是并未以递归来调用。

我们来比较一下两者的区别，试想，如果一个序列只需要递归两次便可结束，即它可以分成四个子序列。原始的方式需要两个递归函数调用，接着两者各自调用一次，也就是说进行了7次函数调用，如下图左边所示。但是STL这种写法每次划分子序列之后仅对右子序列进行函数调用，左边子序列进行正常的循环调用，如下图右边所示。

![img](https://mmbiz.qpic.cn/mmbiz_png/vkC2hqLZ6XHjAV7ywUnnbQCgFIR863C2x1X898OldpQOoPOE3yQaicYSJjvLbxOaSQxjn840ib3hicic4ChhYP6bCw/640?wx_fmt=png)

两者区别就在于STL节省了接近一半的函数调用，由于每次的函数调用有一定的开销，因此对于数据量非常庞大时，这一半的函数调用可能能够省下相当可观的时间。真是为了效率无所不用其极，令人惊叹！更关键是这并没有带来太多的可读性的降低，稍稍一经分析便能够读懂。这种稍稍以牺牲可读性来换取效率的做法在STL的实现中比比皆是，本文后面还会有例子。

### ***三点中值法***

```c++
/// This is a helper function...  
template<typename _RandomAccessIterator, typename _Compare>    
inline _RandomAccessIterator    
__unguarded_partition_pivot(_RandomAccessIterator __first,        _RandomAccessIterator __last, _Compare __comp)    
{   
    //选取中间值      
    _RandomAccessIterator __mid = __first + (__last - __first) / 2;    
    //选取first+1,mid,last-1的三个位置的中间值作为参照物，并存储在first这个位置上     		    //__move_median_to_first函数里为if比较      
    std::__move_median_to_first(__first, __first + 1, __mid, __last - 1,          __comp);     
    //标准的快速排序      
    return std::__unguarded_partition(__first + 1, __last, __first, __comp);    
}
```

取首部、尾部和中部三个元素的中值作为pivot。我们之前学到的快速排序都是选择首部、尾部或者中间位置的元素作为pivot，并不会比较它们的值，在很多情况下这将引起递归的恶化。现在这里采用的中值法可以在绝大部分情形下优于原来的选择。

***分割算法***

```c++
//This is a helper function...  
template<typename _RandomAccessIterator, typename _Compare>    _RandomAccessIterator    
__unguarded_partition(_RandomAccessIterator __first,        _RandomAccessIterator __last,        _RandomAccessIterator __pivot, _Compare __comp)    
{      
    while (true)  
    {    
        while (__comp(__first, __pivot))      
        	++__first;    
     	--__last;    
        while (__comp(__pivot, __last))      
            --__last;    
        if (!(__first < __last))      
            return __first;    
        std::iter_swap(__first, __last);    
        ++__first;  
    }    
}
```

`__unguarded_partition`，这其实就是我们平常所使用的快速排序主体部分，用于根据pivot将区间分割为两个子序列。它会不断去交换放错位置的元素，直到first和last指针相互交错为止，函数返回的是右边区间的起始位置。注意看：这个函数没有对first和last作边界检查，而是以两个指针交错作为中止条件，节约了比较运算的开支。可以这么做的理由是因为，选择是首尾中间位置三个值的中间值作为pivot，因此一定会在超出此有效区域之前中止指针的移动。《STL源码剖析》给出了两个非常直观的示意图：

![img](https://mmbiz.qpic.cn/mmbiz_png/vkC2hqLZ6XHjAV7ywUnnbQCgFIR863C2HYicrWUIpOllcISHXTAiaJSHzMDC8cowdfmlglnExichGYnaG7mEQpfsg/640?wx_fmt=png)

分割示例2

![img](https://mmbiz.qpic.cn/mmbiz_png/vkC2hqLZ6XHjAV7ywUnnbQCgFIR863C26PRAJ9pgDJH07KxdWylHLNtnLOW2zoFvh8W4I8WduHb3ndB9shJUEA/640?wx_fmt=png)

### ***堆排序***

```c++
template<typename _RandomAccessIterator, typename _Compare>    
inline void    
__partial_sort(_RandomAccessIterator __first,       _RandomAccessIterator __middle,       _RandomAccessIterator __last,       _Compare __comp)    
{         
    //建堆      
    std::__heap_select(__first, __middle, __last, __comp);       
    //弹堆      
    std::__sort_heap(__first, __middle, __comp);    
}   
/// This is a helper function for the sort routines.
template<typename _RandomAccessIterator, typename _Compare>    
void    
__heap_select(_RandomAccessIterator __first,      _RandomAccessIterator __middle,      _RandomAccessIterator __last, _Compare __comp)    
{      
    std::__make_heap(__first, __middle, __comp);      
    for (_RandomAccessIterator __i = __middle; __i < __last; ++__i)  
        if (__comp(__i, __first))    
            std::__pop_heap(__first, __middle, __i, __comp);    }
template<typename _RandomAccessIterator, typename _Compare>    
void    
__sort_heap(_RandomAccessIterator __first, _RandomAccessIterator __last,    _Compare __comp)    
{      
    while (__last - __first > 1)  {    
        --__last;    
        td::__pop_heap(__first, __last, __last, __comp);  
    }    
}
```

### ***__final_insertion_sort***

__final_insertion_sort也不是简单的插入排序，STL程序员们在简单的插入排序算法中进一步做了优化，将插入排序又分成了__unguarded_linear_insert，__linear_insert —— 无边界作检查的插入排序和边界作检查的插入排序，不带边界检查的`__unguarded_insertion_sort`更快。

```c++
/// This is a helper function for the sort routine.  
template<typename _RandomAccessIterator, typename _Compare>    
void    
__final_insertion_sort(_RandomAccessIterator __first,         _RandomAccessIterator __last, _Compare __comp)    
{      
    if (__last - __first > int(_S_threshold))  
    {    
        std::__insertion_sort(__first, __first + int(_S_threshold), __comp);                   std::__unguarded_insertion_sort(__first + int(_S_threshold), __last,         __comp);  
    }     
    else  
        std::__insertion_sort(__first, __last, __comp);    
}
```

它被分成了两个分支，前一个分支是处理大于分段阈值的情况，后一个分支处理小于等于分段阈值。第一个问题：为什么要划分成两种情况不同对待？

再看，第一个分支中又将区间分成了两段，前16个和剩余部分，然后分别调用两个排序。于是第二个问题来了，为什么要这么分段？

最后一个问题，`__insertion_sort`和`__unguarded_insertion_sort`有何区别？

***标准插入排序***

摘自维基百科的伪代码

```
for i ← 1 to length(A)    
	j ← i    
	while j > 0 and A[j-1] > A[j]        
			swap A[j] and A[j-1]        
			j ← j - 1
```

从第二个值开始遍历每个元素，首先判断是否有越界，然后判断是否需要交换

![img](https://mmbiz.qpic.cn/mmbiz_gif/vkC2hqLZ6XFjdIV918TTAuQGIFKT3QKORIgibOYG9vndREwEnVwy4Z100LesKpPiaGEfbnsonp16XXu5sczBwslw/640?wx_fmt=gif)

### ***__insertion_sort实现***

```c++
/// This is a helper function for the sort routine.  
template<typename _RandomAccessIterator, typename _Compare>    
void    
__insertion_sort(_RandomAccessIterator __first,         _RandomAccessIterator __last, _Compare __comp)    
{      
    if (__first == __last) return;      
    for (_RandomAccessIterator __i = __first + 1; __i != __last; ++__i){    
        if (__comp(__i, __first)) 
            //会先将该值和第一个元素进行比较，  
            //如果比第一个元素还小,那么就直接将前面已经排列好的数据整体向后移动一位，然后将该元素放在起始位置      
        {        
            typename iterator_traits<_RandomAccessIterator>::value_type    __val = _GLIBCXX_MOVE(*__i);        
            _GLIBCXX_MOVE_BACKWARD3(__first, __i, __i + 1);        
            *__first = _GLIBCXX_MOVE(__val);       
            //对于这种情况，和标准插入排序相比，它将last - first - 1次的比较与交换操作       
            //变成了一次move_backward3操作，节省了每次移动前的比较操作      
        }    
        else //如果该元素并不小于第一个元素，它会调用另外一个函数__unguarded_linear_insert   
            //确保这个区间的左边有效范围内已经有了最小值，否则没有越界检查将可能带来非常严重的后果     
           std::__unguarded_linear_insert(__i,    __gnu_cxx::__ops::__val_comp_iter(__comp));  
    }    
}   
//插入排序，但是少了边界检查    
template<typename _RandomAccessIterator, typename _Compare>    
void    
__unguarded_linear_insert(_RandomAccessIterator __last,            _Compare __comp)    {      
    typename iterator_traits<_RandomAccessIterator>::value_type  __val = _GLIBCXX_MOVE(*__last);      
    _RandomAccessIterator __next = __last;     
    --__next;      
    while (__comp(__val, __next))    
        //这里没有判断是否越界，因为上述调用的分支决定了第一个值在最左边了  
    {    
        *__last = _GLIBCXX_MOVE(*__next);    
        __last = __next;    --__next;  
    }      
    *__last = _GLIBCXX_MOVE(__val);    
}
```

***__unguarded_insertion_sort实现***

```c++
/// This is a helper function for the sort routine. 
template<typename _RandomAccessIterator, typename _Compare>    
inline void    
__unguarded_insertion_sort(_RandomAccessIterator __first,             _RandomAccessIterator __last, _Compare __comp)    
{      
    for (_RandomAccessIterator __i = __first; __i != __last; ++__i) 
        std::__unguarded_linear_insert(__i,         		__gnu_cxx::__ops::__val_comp_iter(__comp));       
    //直接调用了__unguarded_linear_insert    
}
```

此时前面提的最后一个问题：两种插入算法有何区别？已经有了答案：一个带边界检查而另一个不带，不带边界检查的`__unguarded_insertion_sort`更快。

那么为什么不直接使用它呢？这是因为它有一个前提条件，那便是需要确保最小值已经存在于有效区间的最左边。于是，你可能会想，如果此时可以确定最小值已经位于最左边，那么后面所有的区间内便可以使用最快的`__unguarded_insertion_sort`算法。没错，STL的设计者也是这么想的。



可是，如何可以确定最小值已经在最左边了呢？或者在一个小的区间内？绝大部分情况下无法确定。但正是由于快速排序的特殊性，可以保证最小值存在于一个小的区域中，接下来我们会证明这一点。

所以他们想到将经过`__introsort_loop`排序的数据分成两段，假设第一段里面包含了最小值，那么将第一段使用`__insertion_sort`排序，后一段使用`__unguarded_insertion_sort`便可以达到效率的最大化。对，STL的设计者们珍爱效率如生命。

到这里，你可以回答第一个问题了：为什么有这样的分支处理？是因为如果数据量足够小，没有必要进行如此复杂的划分，直接一个插入排序便可以搞定。只数据量比较大的情况下，将数据分成两段，前一段使用带边界检查的插入排序，后一段使用不带边界检查的插入排序。

现在最为关键的一个问题来了，如何可以确保前16个元素中一定有最小值？

### ***论证最小值存在于前16个元素之中***

我们看一下维基百科上快速排序的动画，非常直观：

![img](https://mmbiz.qpic.cn/mmbiz_gif/vkC2hqLZ6XFjdIV918TTAuQGIFKT3QKOjw4Tgjcgk3lFE92whCI5qN51aIyqZacECGUZkXrVC5vxYt7o7nGuLg/640?wx_fmt=gif)

从图中可以看出，无论经过几次递归调用，对于所有划分的区域，左边区间所有的数据一定比右边小，记住这一点，它将为后面的推理起到重要的作用。

再来看一眼`__introsort_loop`：

```c++
template <class RandomAccessIterator, class T, class Size>
void __introsort_loop(RandomAccessIterator first,                      RandomAccessIterator last, T*,                      Size depth_limit) 
{   
	while (last - first > __stl_threshold)
	{        
		if (depth_limit == 0) {           
    		partial_sort(first, last, last);            
    		return;        
    	}        
   		 --depth_limit;        
   		 RandomAccessIterator cut = __unguarded_partition          (first, last, T(__median(*first, *(first + (last - first)/2),                                   *(last - 1))));       
    	__introsort_loop(cut, last, value_type(first), depth_limit);        
    	last = cut;    
    }
 }
```

该函数只有两种情况下可能返回，一是区域小于等于阈值16；二是超过递归深度阈值。我们现在只考虑最左边的子序列，先假设是由于第一种情况终止了这个函数，那么该子区域小于16。再根据前面的结论：左边区间的所有数据一定比右边小，可以推断出最小值一定在该小于16的子区域内。

假设函数是第二种情况下终止，那么对于最左边的区间，由于递归深度过深，因此该区间会调用堆排序，所以这段区间的最小值一定位于最左端。再加上前面的结论：左边区间所有的数据一定比右边小，那么该区间内最左边的数据一定是整个序列的最小值。

因此，不论是哪种情况，都可以保证起始的16个元素中一定有最小值。如此便能够使用`__insertion_sort`对前16个元素进行排序，接着用`__unguarded_insertion_sort`毫无顾忌的在不考虑边界的情况下对剩于的区间进行更快速的排序。

至此，所有三个问题都得到了解答。