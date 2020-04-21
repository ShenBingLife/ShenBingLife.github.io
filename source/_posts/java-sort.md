---
layout: post
title: 排序算法代码总结
category: Java
description: 排序算法代码总结
---


## 排序算法代码总结

### 冒泡排序
```java
	/**
	 * 冒泡排序
	 * 
	 * 原理：
	 * 每一轮循环中依次比较相邻位置上元素的大小，使得较大的元素后移，
	 * 且确保第i轮循环完之后能把第i大的元素移动到排序后的位置上
	 * 
	 * 改进：
	 * 每一轮循环开始前设置标志位，如果本轮循环没有交换任何元素，
	 * 则说明所有元素已经有序，可以提前结束排序
	 * 
	 * @author Roamer
	 */
	void bubbleSort(int[] a) {
	    if(a == null || a.length == 0) return;
	
	    boolean flag;
	    for(int i = a.length - 1; i > 0; --i) {
	        flag = true;//每一轮冒泡之前重置标志位
			for(int j = 0; j < i; ++j) {
	            if(a[j] > a[j+1]) {
	                swap(a, j, j+1);
	                flag = false;
	            }
	        }
	
	        if(flag) break;
	    }
	}
```

### 选择排序
```java
	/**
	 * 选择排序
	 * 
	 * 原理：
	 * 从数组的首元素开始，将第i个元素之后的所有元素通过相互比较找到最小值的索引，
	 * 如果当前元素比这个最小元素大，则交换之，使得第i个位置的元素值为第i小，
	 * 相当于每一轮循环是在所有未排序的元素之中选择出最小的元素。
	 * 
	 * @author Roamer
	 */
	void selectSort(int[] a) {
	    if(a == null || a.length == 0) return;
	
	    for(int i = 0; i < a.length - 1; ++i) {
	        //找到第i个元素之后的最小元素的下标minIdx
			int minIdx = i + 1;
	        for(int j = i + 2; j < a.length; ++j) {
	            if(a[j] < a[minIdx])
	                minIdx = j;
	        }
	        //如果当前元素比其后的最小元素还小，则交换之
			if(a[i] > a[minIdx])
	            swap(a, i, minIdx);
	    }
	}
```

### 插入排序
```java
	/**
	 * 插入排序
	 * 
	 * 原理：
	 * 确保数组前i-1个元素已经排好序，在第i轮循环时，将第i个元素从后往前依次和
	 * 其前面的元素比较和交换，最后插入到前i-1个有序的子数组中的合适位置
	 * 
	 * @author Roamer
	 */
	void insertSort(int[] a) {
	    if(a == null || a.length == 0) return;
	
	    //从数组第二个元素开始进行前插for(int i = 1; i < a.length; ++i) {
	        for(int j = i; j > 0 && a[j-1] > a[j]; --j)
	                swap(a, j-1, j);
	    }
	}
```

### 快速排序

```java
	/**
	 * 快速排序
	 * 
	 * 原理：
	 * 将数组的首元素作为比较的基准，用两个指针从数组的两端往中间扫描，
	 * 当左指针对应的元素大于基准值且右指针对应的元素小于基准值，则交换两者对应的元素值，
	 * 使得每一轮遍历之后数组分成比基准值更大和更小的两部分，
	 * 再把基准元素从数组首位交换到数组的中间，从而其左边的元素都不大于它，
	 * 且其右边的元素都不小于它，然后将左右两个子数组递归调用快排函数，最终使数组有序
	 * 
	 * @author Roamer
	 */
	void quickSort(int[] a, int low, int high) {
	    if(a == null || a.length == 0) return;
	
	    if(low < high){  
	        int pivot = partition(a, low, high);
	        quickSort(a, low, pivot - 1);  
	        quickSort(a, pivot + 1, high);
	    }
	}
	
	//版本一：两个指针互相交换不合格元素（交换之后就都合格了），最后将基准元素移动到中间
	private int partition(int[] a, int low, int high) {
	    //将数组首元素作为每一轮比较的基准int pivot = low;
	
	    while(low < high) {
	        //从右往左扫描，直到遇到比基准元素小的元素
			while(low < high && a[high] >= a[pivot])
	            --high;
	
	        //从左往右扫描，直到遇到比基准元素大的元素
			while(low < high && a[low] <= a[pivot])
	            ++low;
	
	        //将左子数组中不合格的元素与右子数组中不合格的元素交换
	        swap(a, low, high); 
	    }
	
	    //将数组首元素交换到中间位置
	    swap(a, pivot, low);
	
	    //返回数组的中轴位置
		return low;
	}
	
	
	//版本一的改进版：//由于基准元素已经保存了，所以其位置可以被覆盖掉，且两个指针是交替扫描的，
	//所以右边（左边）的不合格元素可以直接覆盖左边（右边）的不合格元素，
	//由于high指针先扫描，然后两个指针的元素交替覆盖对方，所以循环结束后，
	//low对应的位置还没有被覆盖，且它就是两个子数组的分界，将基准元素放到此位置即可
	private int partition1(int[] a, int low, int high) {
	    //将数组首元素作为每一轮比较的基准
	int pivotValue = a[low];
	
	    while(low < high) {
	        //从右往左扫描，直到遇到比基准元素小的元素
			while(low < high && a[high] >= pivotValue)
	            --high;
	
	        //将右子数组中不合格的元素放到左边不合格元素的位置（原元素已经移走）
	        a[low] = a[high];
	
	        //从左往右扫描，直到遇到比基准元素大的元素
			while(low < high && a[low] <= pivotValue)
	            ++low;
	
	        //将左子数组中不合格的元素放到左边不合格元素的位置（原元素已经移走） 
	        a[high] = a[low];
	    }
	
	    //将基准元素放到中间位置
	    a[low] = pivotValue;
	
	    //返回数组的中轴位置return low;
	}
	
	
	//版本二：保持两个指针中总有一个指向基准元素，所以每次交换都是不合格元素与基准元素做交换，
	//当两个指针在数组中间相遇时，low一定指向着基准元素
	private int partition2(int[] a, int low, int high) {
	//将数组首元素作为每一轮比较的基准int pivotValue = a[low];
	
	    while(low < high) {
	        //从右往左扫描，直到遇到比基准元素小的元素
			while(low < high && a[high] >= pivotValue)
	            --high;
	
	        //将右子数组中不合格的元素与基准元素交换
	        swap(a, low, high);
	
	        //从左往右扫描，直到遇到比基准元素大的元素
			while(low < high && a[low] <= pivotValue)
	            ++low;
	
	        //将左子数组中不合格的元素与基准元素交换
	        swap(a, low, high); 
	    }
	
	    //返回数组的中轴位置，low必定指向了基准元素pivotValue
		return low;
	}
```

### 归并排序
```java
	/**
	 * 归并排序（递归版）
	 * 
	 * 原理：
	 * 将数组均分成两个子数组，先将两个子数组分别进行排序，然后合并得到全体元素都有序的数组，
	 * 为使上述的两个子数组分别有序，需要先对其各自的两个子数组进行排序再合并，因此需要递归地
	 * 对每个子数组的两个子数组进行归并排序，直到子数组只有2个元素，此时只需要直接进行合并
	 * 
	 * @author Roamer
	 */
	void MergeSort(int[] a) {
	    if(a == null || a.length == 0) return;
	
	    int[] b = new int[a.length];//辅助数组
	    Merge(a, b, 0, a.length-1, (a.length-1)/2);
	}
	
	//对数组a的两个子数组进行归并排序private void Merge(int[] a, int[] b, int low, int high, int pivot) {
	    //先递归地划分子数组（子数组最小长度为2），并对子数组进行归并排序if(low < high) {
	        Merge(a, b, low, pivot, (low+pivot)/2);
	        Merge(a, b, pivot+1, high, (high+pivot+1)/2);
	    }
	
	    //将已经排好序的两个子数组元素依次进行比较再合并int i = low;
	    int j = pivot+1;
	    int k = low;
	    while(i <= pivot && j <= high){
	        if(a[i] < a[j])
	            b[k++] = a[i++];
	        else
	            b[k++] = a[j++];
	    }
	
	    //取出子数组中可能的剩余元素（每次只可能有一个while执行）while(i <= pivot)  b[k++] = a[i++];
	    while(j <= high)  b[k++] = a[j++];  
	
	    //将本次排好序的部分元素拷贝回原数组a
	    System.arraycopy(b, low, a, low, high-low+1);
	}
	
	
	/**
	 * 归并排序（迭代版）
	 * 
	 * 原理：
	 * 先将整个数组依次划分成若干个长度为2的子数组，对每个子数组中的两个元素进行合并，
	 * 再将整个数组依次划分成若干个长度为4的子数组，对每个子数组中的两个子数组进行合并，
	 * 如此循环，直到只能将数组划分成两个子数组，这两个子数组已经分别有序，直接合并即可
	 * 
	 * @author Roamer
	 *///利用分治策略，对数组a的各级子数组进行迭代归并
	void MergeSort2(int[] a) {
	    if(a == null || a.length == 0) return;
	
	    int[] b = new int[a.length];//辅助数组
		int len = 2;//每一轮合并中数组的长度
		while(len <= a.length) {
	        //将前若干组中两个等长的子数组合并
			int i = 0;
	        while(i + len <= a.length) {
	            Merge2(a, b, i, i+len-1, i+(len-1)/2);
	            i += len;
	        }
	
	        //若原数组长度不是2的幂，则数组可能不能被均分//从而最后一组的两个子数组长度会不同，单独合并之
			if(i != a.length)
	            Merge2(a, b, i, a.length-1, (i+a.length)/2);
	
	        //下一轮合并中数组的长度翻倍
	        len <<= 1;
	
	        //将本轮分组有序的元素拷贝回原数组a
	        System.arraycopy(b, 0, a, 0, a.length);
	    }
	
	    //若原数组长度不是2的幂，需要最后合并一次！
		if(len != a.length) {
	        Merge2(a, b, 0, a.length-1, (len-1)/2);
	        System.arraycopy(b, 0, a, 0, a.length);
	    }
	}
	
	private void Merge2(int[] a, int[] b, int low, int high, int pivot) {
	    //将已经排好序的两个子数组元素依次进行比较再合并int i = low;
	    int j = pivot+1;
	    int k = low;
	    while(i <= pivot && j <= high){
	        if(a[i] < a[j])
	            b[k++] = a[i++];
	        else
	            b[k++] = a[j++];
	    }
	
	    //取出子数组中可能的剩余元素（每次只可能有一个while执行）
	while(i <= pivot)  b[k++] = a[i++];
	    while(j <= high)  b[k++] = a[j++];
	}
```

### 堆排序
```java
	/**
	 * 堆排序
	 * 
	 * 原理：
	 * 先将无序数组构成一个二叉堆（完全二叉树），使得每个节点都小于其子节点（兄弟节点之间可以无序），
	 * 每次取出根节点后，重新调整堆使其仍满足上述特性，每次取出的根节点就构成了一个有序数组。
	 * 
	 * 建堆
	 * 用数组来存放整个二叉堆，且数组的首元素中存储着整个二叉堆的节点总个数，
	 * 建堆时总是在二叉堆的叶子节点处插入新节点，然后“上浮”该节点，最终在数组中得到一个二叉堆。
	 * 
	 * 调整堆
	 * 每次在叶子节点处插入新节点即在数组末位插入新元素需要调整堆，比较新节点和其父节点的大小，
	 * 通过不断交换使其“上浮”，并最终位于二叉树的合适位置；
	 * 每次取出二叉堆的根节点即取出数组的第二个元素后需要调整堆，将二叉树的最后一个叶子节点作为新的根节点，
	 * 然后依次比较其和子节点的大小，通过不断交换使其“下沉”，并最终位于二叉树的合适位置；
	 * 
	 * @author Roamer
	 */
	void heapSort(int[] a) {
	    int[] b = new int[a.length + 1];//二叉堆数组
		//建堆
		for(int i = 0; i < a.length; ++i)
	        insert(b, a[i]);
	
	    //得到有序数组
		for(int i = 0; i < a.length; ++i)
	        a[i] = getRoot(b);
	}
	
	
	//往二叉堆插入新节点，并调整堆
	private void insert(int[] heap, int ele) {
	    heap[0] += 1;//节点总数加1
	    heap[heap[0]] = ele;
	
	    goUp(heap);
	}
	
	//取出二叉堆的根节点，并调整堆private int getRoot(int[] heap) {
	    if (heap[0] < 1) 
	        throw new RuntimeException("二叉堆已经为空，不能再取出元素！");
	
	    int root = heap[1];//取出根节点元素
	    heap[1] = heap[heap[0]];//将二叉堆的最后一个叶子节点作为新的根节点
	    heap[0] -= 1;//节点总数减1
	
	    goDown(heap);
	
	    return root;
	}
	
	//根节点"下沉"
	private void goDown(int[] heap) {
	    int idx = 1;//需要下沉的根节点的索引
		int left, right, minIdx;//minIdx表示左右子节点中较小的那个
		boolean flag = true;//是否有元素交换的标志位
		//如果上一轮循环有元素交换则继续交换
		while(flag) {
	        flag = false;
	        left = (idx << 1);//左子节点
	        right = left + 1;//右子节点if (left > heap[0])//无子节点break; 
	        else if (right > heap[0])//只有左子节点
	            minIdx = left;
	        else
	            minIdx = (heap[left] < heap[right]) ? left : right;
	
	        if (heap[idx] > heap[minIdx]) {
	            swap(heap, idx, minIdx);
	            idx = minIdx;
	            flag = true;//本次循环有元素交换
	        }
	    } 
	}
	
	//末位叶子节点“上浮”private void goUp(int[] heap) {
	    int idx = heap[0];
	    int parent = (idx >> 1);
	
	    while(parent > 0 && heap[idx] < heap[parent]) {
	        swap(heap, idx, parent);
	        idx  = parent;
	        parent = (idx >> 1);
	    }
	}
```

### 希尔排序
```java
	/**
	 * 希尔排序（缩小增量排序）
	 * 
	 * 原理：
	 * 先设置一个较大的步长（增量），将数组分为若干个子序列，对每个子序列分别进行排序，
	 * 再减少步长，再次将数组分为若干个更长的子序列，对每个子序列分别进行排序，
	 * 如此循环，直到步长为1，即整个数组中只有一个子序列，此时整个数组已经有序
	 * 其中，子序列的排序可以采用任何其他排序算法，每一轮排序之后，数组将变得更加有序一些
	 * 
	 * @author Roamer
	 */void shellSort(int[] a) {
	    //得到初始步长int step = 1;
	    while(step < a.length) 
	        step = 3*step + 1;
	
	    while(step > 1) {
	        //缩小步长
	        step = step / 3 + 1;
	        for (int i = 0; i < step; ++i) {
	            // 得到子序列数组int nsub = (a.length - i - 1) / step + 1;
	            int[] sub = new int[nsub];
	            for (int j = 0; j < nsub; ++j)
	                sub[j] = a[i + j * step];
	
	            //对子序列数组进行冒泡排序
	            bubbleSort(sub);
	
	            //将排序后的元素保存到原数组的对应位置for (int j = 0; j < nsub; j++)
	                a[i + j * step] = sub[j];
	        }   
	    }
	}
```

### 基数排序
```java
	/**
	 * 基数排序
	 * 
	 * 原理：
	 * 基数排数基于桶排序的思想。如果需要排序的元素是正整数，则可以通过依次比较他们每个数位上
	 * 数字的大小进行排序，由于十进制只有10个数码，所以只需要10个“桶”就够了。
	 * 基数排序的方式可以采用LSD（Least sgnificant digital）或MSD（Most sgnificant digital），
	 * 如果是采用LSD算法，则从个位开始，将所有整数根据个位数字分别放到对应的10个桶中，
	 * 再按顺序从各个桶中将所有整数取出依次填回原数组，然后将所有整数根据十位数字重新放到新的10个桶中，
	 * 以此类推，直到所有元素的所有数位都遍历完，由于基数排序是稳定的，所以数组中的所有元素最终都有序了
	 * 
	 * @author Roamer
	 */
	void radixSort(int[] a, int len) {
	    int k = 0;
		//用于遍历数组的下标指针
		int m = 1;
		//表示当前用于比较的数位，从个位开始
		int n = 1;//表示数位m对应的权重，即1,10,100,1000...
		//数组的第一维为每个数位上可能出现的数字（0~9），即桶的个数，
		//第二维是包含当前数位的元素可能的总个数，即每个桶中可以放入的整数个数，
		//数组的元素值记录了数位是lsd的整数
		int[][] bucket = new int[10][a.length];
	
	    int[] order = new int[10];//用于记录每个桶中整数的总个数 
		while(m <= len) {
	        //入桶，即将数组中的所有整数按照数位m放到对应的桶中
			for(int i = 0; i < a.length; ++i) {     
	            int lsd = (a[i] / n) % 10;//通过取整取余得到数位m上的数字lsd
	            bucket[lsd][order[lsd]] = a[i];//记录新出现的数位是lsd的整数
	            order[lsd]++;//数位是lsd的整数的个数加1
	        }
	
	
	        //出桶，即从10个桶中依次取出整数，填回数组中（即记录当前已排好的相对顺序）
			for(int i = 0; i < 10; i++) {
	            //如果当前桶不为空
				if(order[i] != 0) {
	                //依次取出当前桶中的记录的整数（bucket数组第二维中元素必定是连续的）
					for(int j = 0; j < order[i]; j++)
	                    a[k++] = bucket[i][j];
	
	                //清空对桶中整数个数的记录 
	                order[i] = 0;
	            }
	        }
	
	        //完成当前数位上的排序，准备下一轮排序
	        k = 0;//将数组a的下标重置为0
	        m++;//数位往高位增加
	        n *= 10;//数位对应的权重增加
	    }
	}
```