## 实现思路

将1个记录插入到已经排好序的有序表中，得到新的、记录数加1的有序表

## 实现细节

```c
void insert_sort(int * list, int length)
{
	// 假设0号位置已经排好序，故从1开始
	for (int i = 1; i < length; i++)
	{
		if (list[i] < list[i - 1])
		{
			// 保存需要找位置进行插入的值，防止元素移位覆盖该元素
			int temp = list[i];
			int j = i - 1;
			for (;j >= 0; j--)
			{
				if (list[j] > temp)
				{
					list[j + 1] = list[j];
				}
				else
				{
					break;
				}
			}
			// 插入到合适的位置
			list[j + 1] = temp;
		}
	}
}
```

