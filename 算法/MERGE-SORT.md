## MERGE-SORT

归并排序是一种经典的分治算法，用于对数组进行排序。它的基本思想是将数组递归地划分为较小的子数组，然后将这些子数组进行合并，最终得到一个有序的数组。

归并排序的代码思路如下：

1. **分解（Divide）**：将待排序的数组递归地分解为较小的子数组，直到子数组的大小为1或为空。
2. **合并（Merge）**：将两个已排序的子数组合并为一个有序的子数组。这个过程中，需要创建一个临时数组来存储合并的结果。
3. **递归（Recursion）**：重复步骤1和步骤2，对左右两个子数组进行递归调用，直到所有的子数组都排序完成。
4. **合并结果**：最后一次合并得到的子数组就是已排序的数组。

<img src="https://images2015.cnblogs.com/blog/1024555/201612/1024555-20161218163120151-452283750.png" alt="img" style="zoom: 25%;" />

具体代码思路如下：

1. 定义一个辅助函数`merge()`，用于合并两个已排序的子数组。函数参数包括原始数组、临时数组、左侧子数组的起始位置`left`、中间位置`mid`和右侧子数组的结束位置`right`。
2. 在`merge()`函数中，设置三个指针：`l_pos`指向左侧子数组的起始位置，`r_pos`指向右侧子数组的起始位置，`pos`指向临时数组的起始位置。
3. 通过比较`arr[l_pos]`和`arr[r_pos]`的大小，将较小的元素放入临时数组`temparr`中，并将对应指针向后移动。
4. 如果某一侧子数组的元素已经全部放入临时数组，而另一侧子数组还有剩余元素，则直接将剩余元素全部放入临时数组。
5. 最后，将临时数组`temparr`中的元素复制回原始数组`arr`的对应位置。
6. 定义一个递归函数`msort()`，用于实现归并排序的分治过程。函数参数包括原始数组、临时数组、左侧子数组的起始位置`left`和右侧子数组的结束位置`right`。
7. 在`msort()`函数中，首先判断如果`left < right`，则继续递归调用`msort()`函数进行划分。
8. 将数组划分为两个子数组后，分别对左右两个子数组进行递归调用`msort()`函数。
9. 最后，调用`merge()`函数将左右两个子数组合并为一个有序的子数组。

```c
#include<stdio.h>
#include<stdlib.h>
void print_arr(int arr[],int n)
{
	for(int i=0;i<n;i++)
	{
		printf("%d ",arr[i]);
	}
	printf("\n");
 } 
 void merge(int arr[],int temparr[],int left,int mid,int right)
 {
 	//临时数组的下标 
 	int l_pos=left;
 	int r_pos=mid+1;
 	int pos=left;
 	//比较并合并
 	while(l_pos<=mid&&r_pos<=right)
 	{
 		if(arr[l_pos]<arr[r_pos])
 			temparr[pos++]=arr[l_pos++];
 		else
 			temparr[pos++]=arr[r_pos++];
	 }
	//防止一边已经放完而另一边没有放入，无法进行比较直接放入即可 
	while(l_pos<=mid)
	{
		temparr[pos++]=arr[l_pos++];
	}
	while(r_pos<=right)
	{
		temparr[pos++]=arr[r_pos++];
	}
	 //将临时数组元素传入原始数组 
	while(left<=right)
	{
		arr[left]=temparr[left];
		left++;
	}
 }
 void msort(int arr[],int temparr[],int left,int right)
 {
 	//如果只有一个元素就不需要划分
	 if(left < right)
	 {
	 	int mid =(left+right)/2;
	 	msort(arr,temparr,left,mid);
	 	msort(arr,temparr,mid+1,right);
	 	merge(arr,temparr,left,mid,right);
	  } 
 }
 
 void merge_sort(int arr[],int n)
 {
 	//分配辅助数组
	 int* temparr = (int*)malloc(n*sizeof(int));
	 if(temparr)
	 {
	 	msort(arr,temparr,0,n-1);
	 	free(temparr);
	  } 
 }
 int main()
 {
 	int arr[] = {9,5,2,7,12,4,3,1,11};
 	int n = 9;
 	print_arr(arr,n);
 	merge_sort(arr,n);
 	print_arr(arr,n);
 	return 0;
 }
```

结果展示：

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230530183603481.png" alt="image-20230530183603481" style="zoom: 67%;" />

归并排序具有稳定性和较好的时间复杂度，适用于对大规模数组进行排序。

## INSERT-SORT

​	插入排序（Insertion Sort）是一种简单直观的排序算法。它的工作方式类似于我们打扑克牌时的排序方式，逐个将未排序的元素插入已排序的子数组中，直到所有元素都被插入并完成排序。

算法思路如下：

通过不断地将当前元素与已排序的子数组进行比较和移动，插入排序逐渐构建起一个有序的数组。

插入排序算法的代码思路如下：
1. 创建一个函数 `insertionSort`，它接受一个数组和数组的长度作为参数。
2. 使用一个循环遍历数组，从第二个元素开始（索引为1），将当前元素视为待插入元素。
3. 在内部循环中，将当前元素与已排序的子数组中的元素进行比较，同时将较大的元素向右移动。
4. 当找到已排序的元素小于或等于当前元素时，将当前元素插入到该位置。
5. 重复步骤2至4，直到所有元素都被插入并完成排序。



<img src="https://lzxhh2017.github.io/2019/03/28/%E6%8F%92%E5%85%A5%E6%8E%92%E5%BA%8F/InsertSort.jpg" alt="插入排序| Mr. Liu's blog" style="zoom:50%;" />

输入和输出部分与快排一致，这里只展示关键部分插入排序算法的代码：

```c++
void insertionSort(int arr[], int n) {
    int i, key, j;
    for (i = 1; i < n; i++) {
        key = arr[i];
        j = i - 1;

        // 将比 key 大的元素向右移动
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j = j - 1;
        }
        arr[j + 1] = key;
    }
}
```

结果展示：

![image-20230530201110299](C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230530201110299.png)

插入排序是一个原地排序算法，它的时间复杂度为**O(n^2)**，其中n是数组的长度。尽管在最坏情况下的时间复杂度较高，但对于小规模的数组或基本有序的数组，插入排序表现良好，并且它是稳定的排序算法，不会改变相等元素的相对顺序。