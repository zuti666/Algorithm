#### 写在前面

上次梳理了10个经典排序算法，今天在看《算法导论》这本书时，看到第7章快速排序。为了弥补假期没学习的遗憾，把第7章学习一下，特此笔记。我在学习的时候越学越吃惊，这本书也太深奥了吧，课后思考题扩展好多，我这个理科生都受不了。原本以为一杯茶，一个算法，整一天。发现不是一天的问题啊，这得整一周才能弄明白啊！怪不得说这本书是神书，忽然感觉头皮有点凉。。。。。。

参考答案来自于https://walkccc.github.io/CLRS/Chap07/Problems/7-3/，感谢大神助力。

# 快速排序(Quick Sort)

对于包含$n$个数的输入数组来说，快速排序是一种最坏情况时间复杂度为$Q(n^2)$的排序算法，但它的平均性能非常好，期望时间复杂度为$Q(nlogn)$，并且$nlogn$中隐含的常数因子非常小。另外它还能进行原址排序，甚至在虚存环境中也能很好工作。

## 快速排序的分治过程

快速排序使用了分治的思想。下面是对一个典型的子数组$A[p,r]$进行快速排序的三步分治过程：

**分解：**

数组A[p,r]被划分为两个（可能为空）子数组$A[p..q-1]$和$A[q+1..r]$，使得$A[p..q-1]$中的每一个元素都小于等于$A[q]$，而$A[q]$也小于等于$A[q+1..r]$中的每一个元素。其中计算下标$q$也是划分过程的一部分。

**解决：**

通过递归调用快速排序，对子数组$A[p..q-1]$和$A[q+1..r]$进行排序。

**合并：**

因为子数组都是原址排序的，所以不需要排序操作：数组$A[p..r]$已经有序。

**伪代码    QuickSort（A,p,r)**

```
QuickSort（A,p,r)
if	p < r
    q = Partition(A,p,r)
    QuickSort(A,p,q-1)
    QuickSort(A,q+1,r)
```

为了排序一个数组$A$的全部元素，初始调用是 QuickSort(A,1,A.length)

## 数组的划分(Partition)

算法的关键部分是Partition过程，它实现了对子数组$A[p..r]$的原址排序。

**伪代码 Partition(A,p,r)**

```
Partition(A,p,r)
x = A[r]
i = p-1
for j = p to r-1
	if A[j] <= x
		i = i + 1
		exchange A[i] with A[j]
exchange A[i+1] with A[r]
return i + 1
```

在第4-7行循环体的每一轮迭代开始时，对于任意数组下标$k$有

1. 若$p \leq k \leq i$，则$A[k] \leq x$。
2. 若$i+1 \leq k \leq j-1$，则$A[k]>x$。
3. 若$k=r$，则$A[k]=x$。

![](E:\Desktop\文章\图片1.jpg)

在子数组$A[p..r]$中，Partition维护了四个区域。$A[p..i]$区间内所有值都小于等于$x$，$A[i+1..j-1]$区间内的所有值都大于$x$，$A[r]=x$。子数组$A[j..r-1]$中的值可能属于任何一种情况。

**循环不变量的证明**

**初始化：**

在循环的第一轮迭代开始之前，$i=p-1,j=p$，并且在$p和i$之间，$i+1和j-1$之间都不存在值，所以循环不变量的前两个条件显然满足。第一行的赋值操作满足了第三个条件。

**保持：**

![](E:\Desktop\文章\IMG_20200224_222632.jpg)

**终止：**

当终止时，$j=r$。我们将数组中的所有元素划分为三个集合：包含了所有小于等于$x$元素的集合、包含了所有大于$x$元素的集合和只有一个元素$x$的集合。

## 执行过程

这里选取了第一个结点作为分隔枢纽。

![](E:\Desktop\文章\quick.gif)

## 快速排序的性能

快速排序的运行时间依赖于划分是否平衡，而平衡与否又依赖于用于划分的元素。

### 最坏情况划分

当划分的两个子问题分别包含了$n-1$个元素和$0$个元素时，快速排序的最坏情况发生了。

不妨假设算法的每一次递归调用中都出现了这种不平衡划分。

划分操作的时间复杂度为$\Theta(n)$。

由于对一个大小为0的数组进行递归调用会直接返回，因此$T(0)=\Theta(1)$。

于是算法运行的递归式可以表示为:
$$
T(n)=T(n-1)+T(0)+\Theta(n)=T(n-1)+\Theta(n)
$$
从直观上，每一层递归累加，结果为$\Theta(n^2)$。实际上，利用代入法也可以得到解为$T(n)=\Theta(n^2)$。

因此，在最坏情况下，快速排序算法的运行时间并不比插入排序好。

此外，当输入数组已经完全有序时，快速排序的时间复杂度仍然为$\Theta(n^2)$，而在同样情况下，插入排序时间复杂度为$O(n)$。

**代入法证明：**

假设$T(n)$是最坏情况下QuickSort在输入规模为$n$的数据集合上花费的时间，则有递归式：
$$
T(n)=max_{0 \leq q \leq n-1}(T(q)+T(n-q-1))+\Theta(n)
$$
因为Partition函数生成的两个子问题的规模总和为$n-1$，所以参数$q$的变化范围是$0 \backsim  {n-1} $。

我们不妨假设$T(n) \leq cn^2$成立，其中$c$为常数。将此式带入上递归式，得：
$$
\begin{aligned}
T(n) & \leq max_{0 \leq q \leq n-1}(cq^2+c(n-q+1)^2+\Theta(n)) \\
     &= c\cdotp max_{0 \leq q \leq n-1}(q^2+(n-q-1)^2)+\Theta(n)
\end{aligned}
$$
表达式$q^2+(n-q+1)^2$在参数取值区间$0 \leq q \leq n-1$的端点上取得最大值。

由于该表达式对$q$的二阶导数是正的，我们可以得到表达式的上界
$$
max_{0 \leq q \leq n-1}(q^2+(n-q+1)^2) \leq (n-1)^2 =n^2-2n+1
$$
将其带入上式中的$T(n)$中，我们得到：

$$
T(n) \leq cn^2-c(2n-1)+\Theta(n) \leq cn^2
$$
因为我们可以选择一个足够大的常数$c$，使得$c(2n-1)$项能显著大于$\Theta(n)$项，所以有$T(n)=O(n^2)$。

### 最好情况划分

在可能的最平衡划分中，Partition得到的两个子问题的规模都不大于$n/2$。这是因为其中一个子问题的规模为$\lfloor n/2 \rfloor$，而另一个子问题的规模为$\lfloor n/2 \rfloor-1$。在这种情况下，快速排序的性能非常好。

此时，算法运行时间的递归式为：$T(n)=2T(n/2)+\Theta(n)$  （我们忽略了一些余项及减1操作的影响）

解得 $T(n)=\Theta(nlogn)$。

### 平衡的划分

快速排序的平均运行时间更接近于其最好情况，而非最坏情况。

事实上，任何一种**常数比例**的划分都会产生深度为$\Theta(logn)$的递归树，其中每一层的时间代价都是$O(n)$。因此只要划分是常数比例的，算法的运行时间总是$O(nlogn)$。

## 快速排序的随机化版本

在讨论快速排序的平均情况时，我们的假设前提是：输入数据的所有排列都是等概率的。但是在实际中，这个假设不会总成立。

采用**随机抽样(random sampling)**的随机化技术。随机抽样是从子数组$A[p..r]$中随机选择一个元素作为主元。为达到这一目的，首先将$A[r]$与从$A[p..r]$中随机选择的一个元素交换位置。通过对序列$p,\cdots，r$的随机抽样，我们可以保证主元元素$x=A[r]$是等概率地从子数组的$r-p+1$个元素中选取的。因为主元元素是随机选取的，我们期望在平均情况下，对输入数组的划分是比较均衡的。

**伪代码Randomized_QuickSort(A,p,r)**

```
Randomized_QuickSort(A,p,r)
if	p < r
	q = Randomized_Partition(A,p,r)
	Randomized_Partition(A,p,q-1)
	Randomized_Partition(A,q+1,r)
```

**Randomized_Partition(A,p,r)**

```
Randomized_QuickSort(A,p,r)
i = Random(p,r)
exchange A[r] with A[i]
return Partition(A,p,r)
```

## 随机化快速排序的期望运行时间

假设待排序的元素始终是互异的

#### *引理   

**当在一个包含$n$个元素的数组上运行QuickSort时，假设在Partition的第5行中所作的比较次数为$X$,那么QuickSort的运行时间为$O(n+X)$**

**证明：**QuickSort的运行时间是在Partition操作所花费的时间决定的。每次对Partition的调用时，都会选择一个主元元素，而且该元素不会被后续的对QuickSort和Partition的递归调用中。因此，在快速排序算法的整个执行阶段，至多只可能调用Partition操作$n$次。每次调用都包含一个固定的工作量和执行若干次$for$循环，在每一次$for$循环中，都要执行第5行。

**我们的目标是计算出$X$,即所有对Partition的调用中，所执行的总的比较次数**。我们并不打算分析在每一次Partition调用中进行了多少次比较，而是希望能够推导出关于总的比较次数的一个界。

为此我们必须了解算法在什么时候对数组中的两个元素进行比较，什么时候不会进行比较。

为了便于分析，我们将**数组A的各个元素重新命名为$z_i,z_{i+1},\cdots ,z_j$，其中$z_i$是数组$A$中的第$i$小的元素。**

**我们还定义$Z_{ij}=\{z_i,z_{i+1},\cdots ,z_j\}$为$z_i$与$z_j$之间（含$i$和$j$）的元素集合。**

算法什么时候会比较$z_i$和$z_j$呢？

我们**首先注意到每一对元素至多比较一次。**因为各个元素只与主元元素进行比较，并且在某一次的Partition调用结束之后，该元素就再也不会与其他元素进行比较了。

我们的分析要用到指示器**随机变量**。定义
$$
X_{ij}=I\{z_i与z_j进行比较\}=\begin{cases}1,如果z_i与z_j进行比较发生 \\ 0,如果z_i与z_j进行比较不发生 \end{cases}
$$
由于**每一对元素至多比较一次**，所以我们可以计算出算法的总比较次数：
$$
X=\sum_{i=1}^{n-1}\sum_{j=i+1}^{n}X_{ij}
$$
对上式两边取期望，再由期望值的线性特征和引理5.1，可以得到：
$$
\begin{aligned}
E(X) &=E[\sum_{i=1}^{n-1}\sum_{j=i+1}^{n}] \\
     &=\sum_{i=1}^{n-1}\sum_{j=i+1}^{n}E[X_{ij}] \\
     &=\sum_{i=1}^{n-1}\sum_{j=i+1}^{n}Pr\{z_i与z_j进行比较\}
\end {aligned}
$$
假设Randomized_Partition随机且独立选择主元。假设每一个元素是互异的。

一旦一个满足$z_i<x<z_j$的主元$x$被选择后，我们就知道$z_i$和$z_j$以后再也不可能进行比较了。

另一种情况，如果$z_i$在$Z_{ij}$中的所有其他元素之前被选择为主元，那么$z_i$就将与除了它自身以外的所有元素进行比较。类似的，如果$z_j$在$Z_{ij}$中的所有其他元素之前被选择为主元，那么$z_j$就将与除了它自身以外的所有元素进行比较。

因此**，$z_i$与$z_j$会进行比较，当且仅当$Z_{ij}$中被选为主元的第一个元素是$z_i$或者$z_j$。**

在$Z_{ij}$的某个元素被选为主元之前，整个集合$Z_{ij}$的元素都属于某一划分的同一分区。因此，$Z_{ij}$中的任何元素都会等可能首先被选为主元。因为集合$Z_{ij}$中有$j-i+1$个元素，并且主元的选取是随机独立的。所以任何元素被首先选为主元的概率是$\frac{1}{j-i+1}$。
$$
\begin{aligned}
Pr\{z_i与z_j进行比较\} & =Pr\{z_i或z_j是集合Z_{ij}中选出的第一个主元 \} \\
				     &=Pr\{z_i是集合Z_{ij}中选出的第一个主元\}+Pr\{z_j是集合Z_{ij}中选出的第一个主元\} \\
				     &=\frac{1}{j=i+1}+\frac{1}{j=i+1} \\
				     &=\frac{2}{j-i+1}
\end{aligned}
$$
上式中第二行成立的原因在于其中涉及的两个事件是互斥的。

于是综上两式我们有:
$$
E[X]=\sum_{i=1}^{n-1}\sum_{j=i+1}^{n}\frac{2}{j-i+1}
$$
在求这个累加和时。可以将变量做个变换$(k=j-i)$，并利用有关调和级数的界，得到：
$$
\begin{aligned}
E[X] &=\sum_{i=1}^{n-1}\sum_{j=i+1}^{n}\frac{2}{j-i+1} \\
     &=\sum_{i=1}^{n-1}\sum_{k=1}^{n-i}\frac{2}{k+1}\\
     &<\sum_{i=1}^{n-1}\sum_{k=1}^{n}\frac{2}{k} \\
     &=\sum_{i=1}^{n-1}O(logn) \\
     &=O(nogn)
\end{aligned}
$$
于是我们可以得到结论：

**使用Randomized_Partition，在输入元素互异的情况下，快速排序的期望运行时间为$O(nlogn)$。**

## 7-3另一种快速排序的分析方法

> An alternative analysis of the running time of randomized quicksort focuses on the expected running time of each individual recursive call to $\text{RANDOMIZED-QUICKSORT}$, rather than on the number of comparisons performed.
>
> 这一方法关注于**每一次单独递归调用的期望运行时间**，而不是比较的次数。

> **a.** **给定一个大小为$n$的数组，特定元素选为主元的概率为$1/n$**。利用这一点来定义**指示器随机变量：**
> $$
> X_{i}=I\{第i小的元素被选为主元\}=\begin{cases}1,如果第i小的元素被选为主元发生 \\ 0,如果第i小的元素被选为主元不发生 \end{cases}
> $$
> $E[X_i]$是什么？
>
> 

> **b.** 设$T(n)$是一个表示快速排序在一个大小为$n$的数组上的运行时间的随机变量，试证明：
> $$
> E[T(n)]=E[\sum_{q=1}^nX_q(T(q-1)+T(n-q)+\Theta(n))]		     \  \  \      	(7.5)
> $$
> 

> **c.** 证明公式$（7.5）$可以重写为：
> $$
> E[T(n)] = \frac{2}{n} \sum_{q=2}^{n-1}E[T(q)]+\Theta(n) \ \ \ (7.6)
> $$

> **d.** 证明：
> $$
> \sum_{k=2}^{n-1}klogk\leq \frac{1}{2}n^2logn-\frac{1}{8}n^2 \  \ (7.7)
> $$
> （提示：可以将该累加式分成两个部分，一部分是$k=2,3,\cdots,\lceil n/2 \rceil-1$，另一部分是$k=\lceil n/2 \rceil ,\cdots,n-1$。）

> **e.** 利用公式$(7.7)$给出的界，证明：公式$(7.6)$中的递归式有解$E[T(n)]=\Theta(nlog(n))$。
>
> （提示：使用代入法，证明对某个正常数$a$和足够大的$n$，有$E[T(n)]\leq anlog(n)$）。

1. **给定一个大小为$n$的数组，特定元素选为主元的概率为$1/n$**。

   定义**指示器随机变量：**
   $$
   X_{i}=I\{第i小的元素被选为主元\}=\begin{cases}1,如果第i小的元素被选为主元发生 \\ 0,如果第i小的元素被选为主元不发生 \end{cases}
   $$
    Since the pivot is selected as a random element in the array, which has size $n$, the probabilities of any particular element being selected are all equal, and add to one, so, are all$ \frac{1}{n}$. . 
   $$
   \begin{aligned}E[X_i] &=Pr\{第i小的元素被选为主元\} \\	   &=\frac{1}{n}\end{aligned}
   $$
   b.**证明：**

    We can apply linearity of expectation over all of the events $X_i$. Suppose we have a particular $X_i$ be true.

   Then, we will have one of the sub arrays be length $i - 1$ and the other be $n - i$,

    and will of course still need linear time to run the partition procedure. 

   This corresponds exactly to the summand in equation $(7.5)$。

   | 被随机选中的元素 | $$z_1$$              | $$z_2$$              | $$\dots$$ | $z_{n-1}$            | $z_n$                |
   | ---------------- | -------------------- | -------------------- | --------- | -------------------- | -------------------- |
   | 运行时间         | $(T(0)+T(n-1)+O(n))$ | $(T(1)+T(n-2)+O(n))$ | $\dots$   | $(T(n-2)+T(1)+O(n))$ | $(T(n-1)+T(0)+O(n))$ |
   | 概率             | $$X_1$$              | $$X_2$$              | $\dots$   | $X_{n-1}$            | $X_{n}$              |

   $$
   T(n)=\sum_{q=1}^nX_q(T(q-1)+T(n-q)+\Theta(n)) \\
   E[T(n)]=E[\sum_{q=1}^nX_q(T(q-1)+T(n-q)+\Theta(n))]	
   $$

   

3. 证明公式$（7.5）$可以重写为：
   $$
   E[T(n)] = \frac{2}{n} \sum_{q=2}^{n-1}E[T(q)]+\Theta(n) \ \ \ (7.6)
   $$
   **证明：**
   $$
   \begin{aligned}
   E[T(n)]& =E[\sum_{q=1}^nX_q(T(q-1)+T(n-q)+\Theta(n))]	 \  \  \     (7.5)\\
          & =\sum_{q=1}^nE[X_q(T(q-1)+T(n-q)+\Theta(n))]  \\
          & =\sum_{q=1}^nE[X_q]*E[(T(q-1)+T(n-q)+\Theta(n))]  \\
          & =\sum_{q=1}^n\frac{1}{n}*E[T(q-1)+T(n-q)+\Theta(n)]  \\
          &= \Theta(n)+\frac{1}{n} * \sum_{q=1}^{n-1}(E[T(q-1)+T(n-q)]) \\ 
          
   \end {aligned}
   $$
   选定$X_q$并完成一次Partition操作后，需要排序的数组剩余长度为n-1,

   被划分的左端和右端继续进行相同的处理，不管怎样划分，由于选取特定元素的概率相同，可以认为在接下来的操作中，左端和右段进行快速排序期望的运行时间相同，不妨令左端等于左端，即$E[T(q-1)]=E[T(n-q)]$,于是我们有
   $$
   \begin{aligned}
   E[T(n)] &= \Theta(n)+\frac{1}{n} * \sum_{q=1}^{n-1}(E[T(q-1)+T(n-q)]) \\ 
           &=\Theta(n)+\frac{1}{n} * \sum_{q=1}^{n-1}(E[T(q-1)+T(q-1)]) \\
           &=\Theta(n)+\frac{2}{n} * \sum_{q=1}^{n-1}(E[T(q-1)]) \\
            &=\Theta(n)+\frac{2}{n} * \sum_{q=2}^{n}(E[T(q)]) \\ 
   \end {aligned}
   $$
   4.证明：
   $$
   \sum_{k=2}^{n-1}klogk\leq \frac{1}{2}n^2logn-\frac{1}{8}n^2 \  \ (7.7)
   $$
   （提示：可以将该累加式分成两个部分，一部分是$k=2,3,\cdots,\lceil n/2 \rceil-1$，另一部分是$k=\lceil n/2 \rceil ,\cdots,n-1$。）
   
   **证明：**
   $$
   \begin{aligned}
   let: \ \    &f(k) =k log(k) \\
               &f'=log(k)+1 \\
             &  if \  k>2, f(k) \nearrow \\
             F &=\int f(k) \\
               &=\frac{x^2}{2}(log(x)-\frac{1}{2})+C \\
              & because \  f(k)>0 , F \nearrow      \\
   \int_2^{n-1} f(k)&=F_{n-1}-F_2 \\
            		 &\leq F_n -F_2 \\
             		 &=\frac{n^2}{2}log(n)-\frac{n^2}{4}-2(log(2)- \frac{1}{2}) \\
             		 &\leq\frac{n^2}{2}log(n)-\frac{n^2}{4}  \\
             		 &\leq\frac{n^2}{2}log(n)-\frac{n^2}{8}   \\
        \sum_{k=2}^{n-1}klogk&\leq \int_2^{n-1} f(k) \\
                        &\leq \frac{n^2}{2}log(n)-\frac{n^2}{8}\\
             		          
             		 
    \end {aligned}
   $$
   5.利用公式$(7.7)$给出的界，证明：公式$(7.6)$中的递归式有解$E[T(n)]=\Theta(nlog(n))$。
   
   （提示：使用代入法，证明对某个正常数$a$和足够大的$n$，有$E[T(n)]\leq anlog(n)$）。
   
   **证明：**
   $$
   \begin{aligned}
   &let: \ \  T(q)\leq qlog(q)+ \Theta(n) \\
   E[T(n)] &= \frac{2}{n} \sum_{q=2}^{n-1}E[T(q)]+\Theta(n) \ \ \ (7.6) \\
         &\leq \frac{2}{n} \sum_{q=2}^{n-1} (qlog(q)+ \Theta(n) )+ \Theta(n)\\               &=\frac{2}{n} \sum_{q=2}^{n-1} qlog(q)+\frac{2}{n}(n-3)\Theta(n)+\Theta(n) \\
         &\leq \frac{2}{n} (\frac{n^2}{2}log(n)-\frac{n^2}{8}) +\Theta(n) \\
         &=nlog(n)-\frac{n}{4}+\Theta(n) \\
         &=nlog(n)+\Theta(n)
         
   
   \end{aligned}
   $$
   

## 7-4快速排序的栈深度

> The $\text{QUICKSORT}$ algorithm of Section 7.1 contains two recursive calls to itself. After $\text{QUICKSORT}$ calls $\text{PARTITION}$, it recursively sorts the left subarray and then it recursively sorts the right subarray. The second recursive call in $\text{QUICKSORT}$ is not really necessary; we can avoid it by using an iterative control structure. This technique, called ***tail recursion\***, is provided automatically by good compilers. Consider the following version of quicksort, which simulates tail recursion:
>
> ```
> TAIL-RECURSIVE-QUICKSORT(A, p, r)
>     while p < r
>         // Partition and sort left subarray.
>         q = PARTITION(A, p, r)
>         TAIL-RECURSIVE-QUICKSORT(A, p, q - 1)
>         p = q + 1
> ```
>
> **a.** Argue that $\text{TAIL-RECURSIVE-QUICKSORT}(A, 1, A.length)$ correctly sorts the array $A$.
>
> Compilers usually execute recursive procedures by using a ***stack\*** that contains pertinent information, including the parameter values, for each recursive call. The information for the most recent call is at the top of the stack, and the information for the initial call is at the bottom. Upon calling a procedure, its information is ***pushed\*** onto the stack; when it terminates, its information is ***popped\***. Since we assume that array parameters are represented by pointers, the information for each procedure call on the stack requires $O(1)$ stack space. The ***stack depth\*** is the maximum amount of stack space used at any time during a computation.
>
> **b.** Describe a scenario in which $\text{TAIL-RECURSIVE-QUICKSORT}$'s stack depth is $\Theta(n)$ on an $n$-element input array.
>
> **c.** Modify the code for $\text{TAIL-RECURSIVE-QUICKSORT}$ so that the worst-case stack depth is $\Theta(\lg n)$. Maintain the $O(n\lg n)$ expected running time of the algorithm.

实际上，在$C++ STL$库中就使用了**尾递归**，这样可以减少栈空间的使用。

它并不是没有管左子序列。注意看，在分割原始区域之后，对左子序列$[p,q-1]$进行了递归，接下来的$p=q+1$将起点位置调整到了分割点，那么此时的区间就是右子序列了。又因为这是一个循环结构，那么在下一次的循环中，右子序列便得到了处理。只是并未以递归来调用。

我们来比较一下两者的区别，试想，如果一个序列只需要递归两次便可结束，即它可以分成四个子序列。原始的方式需要两个递归函数调用，接着两者各自调用一次，也就是说进行了7次函数调用，如下图左边所示。但是STL这种写法每次划分子序列之后仅对右子序列进行函数调用，左边子序列进行正常的循环调用，如下图右边所示。

![](E:\Desktop\文章\比较.png)

 两者区别就在于STL节省了接近一半的函数调用，由于每次的函数调用有一定的开销，因此对于数据量非常庞大时，这一半的函数调用可能能够省下相当可观的时间。 

>**a.** The book proved that $\text{QUICKSORT}$ correctly sorts the array $A$. $\text{TAIL-RECURSIVE-QUICKSORT}$ differs from $\text{QUICKSORT}$ in only the last line of the loop.
>
>It is clear that the conditions starting the second iteration of the **while** loop in $\text{TAIL-RECURSIVE-QUICKSORT}$ are identical to the conditions starting the second recursive call in $\text{QUICKSORT}$. Therefore, $\text{TAIL-RECURSIVE-QUICKSORT}$ effectively performs the sort in the same manner as $\text{QUICKSORT}$. Therefore, $\text{TAIL-RECURSIVE-QUICKSORT}$ must correctly sort the array $A$.
>
>**b.** The stack depth will be $\Theta(n)$ if the input array is already sorted. The right subarray will always have size $0$ so there will be $n − 1$ recursive calls before the **while**-condition $p < r$ is violated.\
>
>**c.**
>
>```
>MODIFIED-TAIL-RECURSIVE-QUICKSORT(A, p, r)
>    while p < r
>        q = PARTITION(A, p, r)
>        if q < floor((p + r) / 2)
>            MODIFIED-TAIL-RECURSIVE-QUICKSORT(A, p, q - 1)
>            p = q + 1
>        else
>            MODIFIED-TAIL-RECURSIVE-QUICKSORT(A, q + 1, r)
>            r = q - 1
>```
>
>

## 7-5三数取中划分

> One way to improve the $\text{RANDOMIZED-QUICKSORT}$ procedure is to partition around a pivot that is chosen more carefully than by picking a random element from the subarray. One common approach is the ***median-of-3\***method: choose the pivot as the median (middle element) of a set of 3 elements randomly selected from the subarray. (See exercise 7.4-6.) For this problem, let us assume that the elements of the input array $A[1..n]$ are distinct and that $n \ge 3$. We denote the sorted output array by $A'[1..n]$. Using the median-of-3 method to choose the pivot element $x$, define $p_i = \Pr\{x = A'[i]\}$.
>
> **a.** Give an exact formula for $p_i$ as a function of $n$ and $i$ for $i = 2, 3, \ldots, n - 1$. (Note that $p_1 = p_n = 0$.)
>
> **b.** By what amount have we increased the likelihood of choosing the pivot as $x = A'[\lfloor (n + 1) / 2 \rfloor]$, the median of $A[1..n]$, compared with the ordinary implementation? Assume that $n \to \infty$, and give the limiting ratio of these probabilities.
>
> **c.** If we define a "good" split to mean choosing the pivot as $x = A'[i]$, where $n / 3 \le i \le 2n / 3$, by what amount have we increased the likelihood of getting a good split compared with the ordinary implementation? ($\textit{Hint:}$ Approximate the sum by an integral.)
>
> **d.** Argue that in the $\Omega(n\lg n)$ running time of quicksort, the median-of-3 method affects only the constant factor.





>**a.** $p_i$ is the probability that a randomly selected subset of size three has the $A'[i]$ as it's middle element. There are 6 possible orderings of the three elements selected. So, suppose that $S'$ is the set of three elements selected. We will compute the probability that the second element of $S'$ is $A'[i]$ among all possible $3$-sets we can pick, since there are exactly six ordered $3$-sets corresponding to each $3$-set, these probabilities will be equal. We will compute the probability that $S'[2] = A[i]$. 
>
>For any such $S'$, we would need to select the first element from $[i - 1]$ and the third from ${i + 1, \ldots , n}$. So, there are $(i - 1)(n - i)$ such $3$-sets. 
>
>The total number of $3$-sets is $\binom{n}{3} = \frac{n(n - 1)(n - 2)}{6}$. 
>
>So,$$p_i = \frac{6(n - i)(i - 1)}{n(n - 1)(n - 2)}.$$
>
>**b.** If we let $i = \lfloor \frac{n + 1}{2} \rfloor$, the previous result gets us an increase of
>
>$$\frac{6(\lfloor\frac{n - 1}{2}\rfloor)(n - \lfloor\frac{n + 1}{2}\rfloor)}{n(n - 1)(n - 2)} - \frac{1}{n}$$
>
>in the limit $n$ going to infinity, we get  $$\lim_{n \to \infty} \frac{\frac{6(\lfloor \frac{n - 1}{2} \rfloor)(n - \lfloor \frac{n + 1}{2} \rfloor)}{n(n - 1)(n - 2)}}{\frac{1}{n}} = \frac{3}{2}.$$
>
>
>
>**c.** To save the messiness, suppose $n$ is a multiple of $3$. We will approximate the sum as an integral, so,
>
>$$ \begin{aligned} \sum_{i = n / 3}^{2n / 3} & \approx \int_{n / 3}^{2n / 3} \frac{6(-x^2 + nx + x - n)}{n(n - 1)(n - 2)}dx \\ & = \frac{6(-7n^3 / 81 + 3n^3 / 18 + 3n^2 / 18 - n^2 / 3)}{n(n - 1)(n - 2)}, \end{aligned} $$
>
>which, in the limit $n$ goes to infinity, is $\frac{13}{27}$ which is a constant that $>\frac{1}{3}$ as it was in the original randomized quicksort implementation.
>
>**d.** Even though we always choose the middle element as the pivot (which is the best case), the height of the recursion tree will be $\Theta(\lg n)$. Therefore, the running time is still $\Omega(n\lg n)$.



## 7-6对区间的模糊排序

>Consider the problem in which we do not know the numbers exactly. Instead, for each number, we know an interval on the real line to which it belongs. That is, we are given $n$ closed intervals of the form $[a_i, b_i]$, where $a_i \le b_i$. We wish to ***fuzzy-sort\*** these intervals, i.e., to produce a permutation $\langle i_1, i_2, \ldots, i_n \rangle$ of the intervals such that for $j = 1, 2, \ldots, n$, there exists $c_j \in [a_{i_j}, b_{i_j}]$ satisfying $c_1 \le c_2 \le \cdots \le c_n$.
>
>**a.** Design a randomized algorithm for fuzzy-sorting $n$ intervals. Your algorithm should have the general structure of an algorithm that quicksorts the left endpoints (the $a_i$ values), but it should take advantage of overlapping intervals to improve the running time. (As the intervals overlap more and more, the problem of fuzzy-sorting the intervals becoes progressively easier. Your algorithm should take advantage of such overlapping, to the extend that it exists.)
>
>**b.** Argue that your algorithm runs in expected time $\Theta(n\lg n)$ in general, but runs in expected time $\Theta(n)$ when all of the intervals overlap (i.e., when there exists a value $x$ such that $x \in [a_i, b_i]$ for all $i$). Your algorithm should not be checking for this case explicitly; rather, its performance should naturally improve as the amount of overlap increases.



>**a.** With randomly selected left endpoint for the pivot, we could trivially perform fuzzy sorting by quicksorting the left endpoints, $a_i$'s. This would achieve the worst-case expected running time of $\Theta(n\lg n)$. We definitely can do better by exploit the characteristic that we don't have to sort overlapping intervals. That is, for two overlapping intervals, $[a_i, b_i]$ and $[a_j, b_j]$. In such situations, we can always choose $\{c_i, c_j\}$ (within the intersection of these intervals) such that $c_i \le c_j$ or $c_j \le c_i$.
>
>Since overlapping intervals do not require sorting, we can improve the expected running time by modifying quicksort to identify overlaps:
>
>```
>FIND-INTERSECTION(A, p, r)
>    rand = RANDOM(p, r)
>    exchange A[rand] with A[r]
>    a = A[r].a
>    b = A[r].b
>    for i = p to r - 1
>        if A[i].a ≤ b and A[i].b ≥ a
>            if A[i].a > a
>                a = A[i].a
>            if A[i].b < b
>                b = A[i].b
>    return (a, b)
>```
>
>
>
>On lines 2 through 3 of $\text{FIND-INTERSECTION}$, we select a random *pivot interval* as the initial region of overlap $[a ,b]$. There are two situations:
>
>- If the intervals are all disjoint, then the estimated region of overlap will be this randomly-selected interval;
>- otherwise, on lines 6 through 11, we loop through all intervals in arrays $A$ (except the endpoint which is the initial pivot interval). At each iteration, we determine if the current interval overlaps the current estimated region of overlap. If it does, we update the estimated region of overlap as $[a, b] = [a_i, b_i] \cap [a, b]$.
>
>$\text{FIND-INTERSECTION}$ has a worst-case running time $\Theta(n)$ since we evaluate the intersection from index $1$ to $A.length$ of the array.
>
>We can extend the $\text{QUICKSORT}$ to allow fuzzy sorting using $\text{FIND-INTERSECTION}$.
>
>First, partition the input array into "left", "middle", and "right" subarrays. The "middle" subarray elements overlap the interval $[a, b]$ found by $\text{FIND-INTERSECTION}$. As a result, they can appear in any order in the output.
>
>We recursively call $\text{FUZZY-SORT}$ on the "left" and "right" subarrays to produce a fuzzy sorted array in-place. The following pseudocode implements these basic operations. One can run $\text{FUZZY-SORT}(A, 1, A.length)$ to fuzzy-sort an array.
>
>The first and last elements in a subarray are indexed by $p$ and $r$, respectively. The index of the first and last intervals in the "middle" region are indexed by $q$ and $t$, respectively.
>
>```
>FUZZY-SORT(A, p, r)
>    if p < r
>        (a, b) = FIND-INTERSECTION(A, p, r)
>        t = PARTITION-RIGHT(A, a, p, r)
>        q = PARTITION-LEFT(A, b, p, t)
>        FUZZY-SORT(A, p, q - 1)
>        FUZZY-SORT(A, t + 1, r)
>```
>
>
>
>We need to determine how to partition the input arrays into "left", "middle", and "right" subarrays in-place.
>
>First, we $\text{PARTITION-RIGHT}$ the entire array from $p$ to $r$ using a pivot value equal to the left endpoint $a$ found by $\text{FIND-INTERSECTION}$, such that $a_i \le a$.
>
>Then, we $\text{PARTITION-LEFT}$ the subarray from $p$ to $t$ using a pivot value equal to the right endpoint $b$ found by $\text{FIND-INTERSECTION}$, such that $b_i < b$.
>
>```
>PARTITION-RIGHT(A, a, p, r)
>    i = p - 1
>    for j = p to r - 1
>        if A[j].a ≤ a
>            i = i + 1
>            exchange A[i] with A[j]
>    exchange A[i + 1] with A[r]
>    return i + 1
>PARTITION-LEFT(A, b, p, t)
>    i = p - 1
>    for j = p to t - 1
>        if A[j].b < b
>            i = i + 1
>            exchange A[i] with A[j]
>    exchange A[i + 1] with A[t]
>    return i + 1
>```
>
>
>
>The $\text{FUZZY-SORT}$ is similar to the randomized quicksort presented in the textbook. In fact, $\text{PARTITION-RIGHT}$ and $\text{PARTITION-LEFT}$ are nearly identical to the $\text{PARTITION}$ procedure on page 171. The primary difference is the value of the pivot used to sort the intervals.
>
>**b.** We expect $\text{FUZZY-SORT}$ to have a worst-case running time $\Theta(n\lg n)$ for a set of input intervals which do not overlap each other. First, notice that lines 2 through 3 of $\text{FIND-INTERSECTION}$ select a *random interval* as the initial pivot interval. Recall that if the intervals are disjoint, then $[a, b]$ will simply be this initial interval.
>
>Since for this example there are no overlaps, the "middle" region created by lines 4 and 5 of $\text{FUZZY-SORT}$ will only contain the initially-selected interval. In general, line 3 is $\Theta(n)$. Fortunately, since the pivot interval $[a, b]$ is randomly-selected, the expected sizes of the "left" and "right" subarrays are both $\left\lfloor \frac{n}{2} \right\rfloor$. In conclusion, the recurrence of the running time is
>
>$$ \begin{aligned} T(n) & \le 2T(n / 2) + \Theta(n) \\ & = \Theta(n\lg n). \end{aligned} $$
>
>The $\text{FIND-INTERSECTION}$ will always return a non-empty region of overlap $[a, b]$ containing $x$ if the intervals all overlap at $x$. For this situation, every interval will be within the "middle" region since the "left" and "right" subarrays will be empty, lines 6 and 7 of $\text{FUZZY-SORT}$ are $\Theta(1)$. As a result, there is no recursion and the running time of $\text{FUZZY-SORT}$ is determined by the $\Theta(n)$ running time required to find the region of overlap. Therefor, if the input intervals all overlap at a point, then the expected worst-case running time is $\Theta(n)$.

