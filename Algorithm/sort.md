### 排序算法

- 冒泡排序

  思想：从第一个数字开始，与下一位数字进行比较，如果大，交换位置，如果小，继续，重复步骤，直到最后一个数字即可进行正确的排序

  ```java
  	public static int[] bubbleSort(int[] arr) {
  		int length = arr.length;
  		for(int i=0; i < length-1; i++) {
  			for(int j=0; j < length-i-1; j++) {
  				if(arr[j] > arr[j+1]) {
  					int temp = arr[j];
  					arr[j] = arr[j+1];
  					arr[j+1] = temp;
  				}
  			}
  		}
  		return arr;
  	}
  ```

  最好的情况是数组已经完全有序，此时算法的时间复杂度为n，否则其他的情况时间复杂度为n^2；

  优化：

- 选择排序

  思想：第一个数字与之后的每一个数字进行比较，比较一次后即可找出最小的数字，第二趟，使得第二个数字与之后的每一个数字进行比较，进行n-1次后即可

  ```java
  public static int[] selectSort(int[] arr) {
  		int length = arr.length;
  		for(int i=0; i<length-1; i++) {
  			int min = i;
  			for(int j=i+1; j<length; j++) {
  				if(arr[min] > arr[j])
  					min = j;
  			}
  			int temp = arr[min];
  			arr[min] = arr[i];
  			arr[i] = temp;
  		}
  		return arr;
  	}
  ```

  时间复杂度为n^2

- 快速排序

  是中大规模数据排序的首选，其时间复杂度是nlogN,最坏的时间复杂度会降为n^2

  思想：选取一个切分元素，而后数据元素的交换使得整个元素处于某个位置，使得满足左边的数字都小于等于它，右边的数字都大于等于它

  ```java
  //快速排序
  	public static int[] quickSort(int[] arr, int left, int right) {
  		if(left < right) {
  			int mid = partion(arr, left, right);
  			quickSort(arr, 0, mid-1);
  			quickSort(arr, mid+1, right);
  		}
  		return arr;
  	}
  	//切分元素选取
  	public static int partion(int[] arr, int left, int right) {
  		while(left < right) {
  			while(left < right && arr[left] <= arr[right]) {
  				left++;
  			}
  			if(left < right) {
  				int temp = arr[left];
  				arr[left] = arr[right];
  				arr[right] = temp; 
  			}
  			while(left < right && arr[left] <= arr[right]) {
  				right--;
  			}
  			if(left < right) {
  				int temp = arr[left];
  				arr[left] = arr[right];
  				arr[right] = temp; 
  			}
  		}
  		return left;
  	}
  ```

- 希尔排序

  思想：对原先的数据进行分组，然后做直接插入排序，完成后缩小数组的数量，重复操作，直到变为一个数组

  算法的好坏取决于段的选取，对于不同的数据量选取不同的段数；

  时间复杂度为nlogN，最坏的时间复杂度是n^2；

  ~~~java
      public static void shellSort(int[] arr) {
          int j, tmp;
          for (int sec = arr.length/2; sec > 0; sec/=2) {
              for (int i = sec; i < arr.length; i++) {
                  tmp = arr[i];
                  for (j = i - sec; j >= 0; j -= sec) {
                      if (tmp < arr[j]) {
                          arr[j + sec] = arr[j];
                      } else {
                          break;
                      }
                  }
                  arr[j + sec] = tmp;
              }
          }
      }
  ~~~

- 归并排序

  思想：采用分治的思想，把一个大问题分解为多个小问题，解决了每一个小问题，后把小问题的解合并就得到大问题的解，时间复杂度为nlogN

  ```java
  	public static void mergeSort2(int[]arr, int low, int mid, int high) {
  		int[] temp = new int[arr.length];
  		int i = low, j = mid+1, k = low;
  		while(i <= mid && j <= high) {
  			if(arr[i] < arr[j])
  				temp[k++] = arr[i++];
  			else
  				temp[k++] = arr[j++];
  		}
  		while(i <= mid) {
  			temp[k++] = arr[i++];
  		}
  		while(j <= high) {
  			temp[k++] = arr[j++];
  		}
  		for(int l=low; l<=high; l++) {
  			arr[l] = temp[l];
  		}
  	}
  	//归并排序(二路归并)
  	public static void mergeSort(int[] arr, int low, int high) {
  		if(low < high) {
  			int mid = low + (high-low)/2;
  			mergeSort(arr, low, mid);
  			mergeSort(arr, mid+1, high);
  			mergeSort2(arr, low, mid, high);
  		}
  	}
  ```

- 堆排序

  利用堆的数据结构的性质设计的算法，是一个近似于完全的二叉树，子节点的值用于大于或者小于父节点；

  思想：初始时时无序区，把堆顶元素与最后一个元素互换后，可能会违反堆的性质，因此需要调整为新堆，不断的重复此过程，即可完成排序；

  ~~~java
      public static void heapSort(int[] arr) {
          //初始化堆
          for (int i = arr.length/2; i >= 0; i--) {
             adjust(arr, i, arr.length);
          }
          for (int i = arr.length-1; i > 0; i--) {
              swap(arr, 0, i); //交换操作
              adjust(arr, 0, i); //调整操作
          }
      }
      //创建堆的函数
      public static void adjust(int[] arr, int i, int len) {
          int child;
          int father;
          for (father = arr[i]; getLeft(i) < len; i = child) {
              child = getLeft(i);
              if (child != len -1 && arr[child] < arr[child+1]) {
                  child++;
              }
              if (father < arr[child]) {
                  arr[i] = arr[child];
              } else {
                  break;
              }
          }
          arr[i] = father;
      }
      //得到节点的左孩子
      public static int getLeft(int i) {
          return i * 2 + 1;
      }
      //交换数字
      public static void swap(int[] arr, int left, int right) {
          int temp = arr[left];
          arr[left] = arr[right];
          arr[right] = temp;
      }
  ~~~

- 直接插入排序

  ~~~java
      public void insertionSort(int[]a) {
         for(int i=0;i<a.length-1;i++)
         {
             int currentItem=a[i+1];
             for(int j=i;j>=0;j--)
             {
                 if(currentItem<a[j]) {
                     a[j+1]=a[j];
                     a[j]=currrentItem;
                 }
                 else {
                     a[j+1]=currentItem;
                     break;
                 }
             }
         }
      }
  ~~~

  

- 二分查找法

  面试必考：必须掌握

  ```java
  	public static boolean binarySearch(int[] arr, int num) {
  		int low = 0;
  		int high = arr.length - 1;
  		while(low <= high) {
  			int mid = (low + high) / 2;
  			if(arr[mid] == num) 
  				return true;
  			else if (arr[mid] < num) {
  				low = mid + 1;
  			}  else {
  				high = mid - 1;
  			}
  		}
  		return false;
  	}
  ```

  