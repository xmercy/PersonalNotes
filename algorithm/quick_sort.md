## 实现思路

每一趟排序，选取一个枢纽值，将数组元素划分为小于枢纽值和大于枢纽值两部分，等价于子数组，然后递归的对每一部分再次进行相同的排序。当需要排序的子数组元素个数为1时，递归调用返回。

## 实现细节

```c
/*
	三数取中法，选取中值作为枢纽值，
	并将中值放在low位置，返回枢纽值
*/
int select_pivot(int * list, int low, int high)
{
	int middle = (low + high) / 2;
	if (list[low] > list[high])
	{
		swap_integer(&list[low], &list[middle]);
	}
	if (list[middle] > list[high])
	{
		swap_integer(&list[middle], &list[high]);
	}
	if (list[middle] > list[low])
	{
		swap_integer(&list[low], &list[middle]);
	}
	return list[low];
}

/*
	选取枢纽值，放置于low位置，左右交替与枢纽值比较
*/
int partition(int * list, int low, int high)
{
	int pivot_value = select_pivot(list, low, high);
	while (low < high)
	{
		// 大于等于防止待排数组中有与枢纽值相同元素时死循环
		while (low < high && list[high] >= pivot_value)
		{
			high--;
		}
		list[low] = list[high];
		while (low < high && list[low] <= pivot_value)
		{
			low++;
		}
		list[high] = list[low];
	}
	list[low] = pivot_value;
	return low;
}

void quick_sort(int * list, int low, int high)
{
	if (high - low <= 10)
	{
		// 小数组使用采用插入排序，减少递归
		insert_sort(list + low, high - low + 1);
	}
	else
	{
		if (low < high)
		{
			// 选择枢纽值，对数组中的元素按枢纽值分区
			int pivot_idx = partition(list, low, high);
			quick_sort(list, low, pivot_idx - 1);
			quick_sort(list, pivot_idx + 1, high);
		}
	}
}
```

