---
title: 快速排序

date: 2017-05-09 20:05

updated: 2017-05-09 20:05

tags:
- 算法

categories: Java

permalink: quick-sort
---

## 简介

这篇POST是**快速排序**的DEMO。

在原始的快速排序追加了一些增强：

- 当递归进行到后期，待排序数组足够小时，剩余元素使用插入排序（InsertionSort）效率更好
- 使用**三数取中法**作为算法的第一步，用于获取基准元素。



## 源码

~~~java
public class QuickSort<T extends Comparable<? super T>> {
    
    /**
     * 这是个经验值。当待排序数组小于等于这个值时，对剩余元素进行插入排序
     */
    private static final int CUTOFF = 10; 
    
    public void sort(T[] arr) {
        quickSort(arr, 0, arr.length - 1);
    }
    
    /**
     * 算法主体 快速排序
     * 这个方法会不断地递归自身，一次递归代表“一趟排序”
     */
    private void quickSort(T[] arr, int left, int right) {
        int size = right - left;
        if (size > CUTOFF) {
            T pivot = median3(arr, left, right);
            
            // 定义游标i, j，由于最左和最右的元素经过三数取中法之后
            // 已经确保了前者小于基准元素，后者大于基准元素
            // 所以游标应该越过他们
            int i = left;
            int j = right - 1;
            
            // 游标开始移动，直到i不在j的左侧
            for(;;) {
                // 游标i不断向右侧移动，直到所指向的元素不再小于基准元素，才停止
                while (arr[++i].compareTo(pivot) < 0) {}
                // 游标j不断向左侧移动，直到所指向的元素不再大于基准元素，才停止
                while (arr[--j].compareTo(pivot) > 0) {}
                
                // i还在j的左侧，而且游标各自指向一个小于基准和大于基准的元素
                if (i < j) {
                    // 大于基准的元素换到右侧，小于基准的元素换到左侧
                    swapRef(arr, i, j);
                    
                // 这种情况指的是i, j擦身而过才停止，那么i已经进入了j曾走过的路径，j也如此，说明小于基准元素的元素和大于基准元素的元素已经分离，分离点正是i
                } else {
                    // 既然已经分离了，那么游标的工作就结束了
                    break;
                }
            }
            // 将i指向的元素，和基准元素互换，达到以下示意的效果
            // [  *小于基准的元素们*  ]， [基准元素]， [  *大于基准的元素们*  ]
            swapRef(arr, i, right - 1);
            
            
            // 以基准元素为分割，前后递归分治
            quickSort(arr, left, i - 1);
            quickSort(arr, i + 1, right);
            
        } else {
            insertionSort(arr, left, right);
        }
    }
    
    /** 三分法取基准元素 */
    private T median3(T[] arr, int left, int right) {
        int center = (left + right) / 2;
        // 1、确保最左元素小于中间的元素
        if (arr[left].compareTo(arr[center]) > 0) {
            swapRef(arr, left, center);
        }
        // 2、确保最左元素小于最右元素（结合1、2，最左元素是最小的了）
        if (arr[left].compareTo(arr[right]) > 0) {
            swapRef(arr, left, right);   
        }
        // 3、确保中间元素大于最右元素（结合2、3，最右元素是最大的了）（结合2、3括号里的内容，中间元素是中间值）
        if (arr[center].compareTo(arr[right]) > 0) {
            swapRef(arr, center, right);   
        }
        // 基准元素移动到倒数第二位，为了不阻挡后续游标i, j的移动
        swapRef(arr, center, right - 1);
        return arr[right - 1];
    }
    
    /**
     * 插入排序
     * 对数组从left到right的元素进行排序
     */
    private void insertionSort(T[] arr, int left, int right) {
        int j;
        
        // 这里一次循环代表"一趟排序"
        // “第i趟排序”结束时，从left至第i号的元素是有序的，所以i从left+1开始，到right为止
        for (int i = left + 1; i <= right; i++) {
            // 每趟都是为i号元素进行插队
            T selected = arr[i];
  			for (j = i; j > 0 && arr[j - 1].compareTo(selected) > 0; j--) {  
                // 准备插队的过程中，比插队元素大的元素应该往后走一位，效果就是一个“空位”不断向左移
                arr[j] = arr[j - 1];
            }
            // “空位”停止了移动，因为“空位”前方的元素比插队元素小，
            // 而“空位”后方也是比插队元素大的最后一个元素，
            // 插队元素完成了插队，该趟插入排序结束
            arr[j] = selected;
        }
    }

    /** 位置交换 */
    private void swapRef(T[] arr, int oneIndex, int anotherIndex) {
        T temp = arr[oneIndex];
        arr[oneIndex] = arr[anotherIndex];
        arr[anotherIndex] = temp;
    }

}
~~~



## 测试

~~~java
	public static void main(String[] args) {
        // 生成一个长度为100，元素在0～50随机分布的列表
        Random random = new Random();
        List<Integer> list = new ArrayList<>();
        for (int i = 0; i < 100; i++) {
            list.add(random.nextInt(50));
        }
        System.out.println(list);

        System.out.println("");
        
        Integer[] arr = list.toArray(new Integer[0]);
        QuickSort<Integer> sorter = new QuickSort<>();
        sorter.sort(arr);
        for (Integer ele : arr) {
            System.out.print(ele + "　");
        }
    }
~~~



~~~
[29, 28, 28, 14, 17, 35, 27, 18, 28, 17, 44, 3, 7, 33, 39, 19, 36, 2, 8, 42, 48, 47, 31, 44, 46, 40, 24, 26, 31, 34, 2, 8, 19, 29, 25, 26, 28, 20, 6, 20, 28, 5, 23, 37, 16, 17, 25, 30, 34, 44, 9, 20, 27, 23, 4, 10, 17, 26, 5, 8, 0, 3, 32, 20, 2, 30, 47, 44, 10, 20, 17, 23, 42, 14, 13, 1, 35, 29, 39, 42, 17, 1, 16, 19, 14, 36, 9, 15, 7, 9, 1, 1, 47, 14, 48, 10, 36, 35, 17, 34]

0　1　1　1　1　2　2　2　3　3　4　5　5　6　7　7　8　8　8　9　9　9　10　10　10　13　14　14　14　14　15　16　16　17　17　17　17　17　17　17　18　19　19　19　20　20　20　20　20　23　23　23　24　25　25　26　26　26　27　27　28　28　28　28　28　29　29　29　30　30　31　31　32　33　34　34　34　35　35　35　36　36　36　37　39　39　40　42　42　42　44　44　44　44　46　47　47　47　48　48　
~~~

