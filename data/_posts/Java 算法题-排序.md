---
title: Java 算法题-排序
date: 2017-07-10 10:02:36
tags: [算法,排序,Java]
categories: Java 算法题
---

# 交换排序

<!-- more -->

## 冒泡排序

```
public class BubbleSort {

    public static void bubbleSort(int[] a) {
        if (null == a)
            throw new NullPointerException("a is null");
        for (int i = 0; i < a.length - 1; i++) {
            for (int j = 0; j < a.length - 1 - i; j++) {
                if (a[j] > a[j + 1]) swap(a, j, j + 1);
            }
        }
    }

    private static void swap(int[] a, int i, int j) {
        int temp = a[i];
        a[i] = a[j];
        a[j] = temp;
    }
}
```

第1层：确定每次冒泡的边界，索引依次为a.length - 1, a.length - 2, ..., 1。不妨将这个边界设为r。一共需进行a.length - 1次。每一次大轮的排序都会确保边界位置填入正确的值。

第2层：在每个边界内部，通过不断的依序两两交换将索引[0,r]中的最大值沉底入索引r。

例如：

```
索引： 0    1    2    3    4
数值： 5    4    3    2    1
```

第1层，确定每次冒泡的边界，索引依次为4, 3, 2, 1。一共需进行4次。

第2层：

第1次结束：4, 3, 2, 1, 5。

第2次结束：3, 2, 1, 4, 5。

第3次结束：2, 1, 3, 4, 5。

第4次结束：1, 2, 3, 4, 5。

在代码中：

i的取值范围为[0,a.length-2]。代表冒泡需要进行的次数，其作用仅仅是为了计数：共需进行a.length - 1次冒泡。

a.length - 1 - i的取值范围为[a.length - 1, 1]。代表每次冒泡时的边界r。

## 快速排序

```
public class QuickSort {

    public static void quickSort(int[] a) {
        if (null == a)
            throw new NullPointerException("a is null");
        quickSort(a, 0, a.length - 1);
    }

    private static void quickSort(int[] a, int low, int high) {
        if (low >= high) return;
        int keyIndex = partition(a, low, high);
        quickSort(a, low, keyIndex - 1);
        quickSort(a, keyIndex + 1, high);
    }

    private static int partition(int[] a, int low, int high) {
        // left的使命是扫描出比轴大的元素
        // right的使命是扫描出比轴小的元素
        int left = low, right = high;
        int keyValue = a[low];
        while (left < right) {
            while (right > left && a[right] >= keyValue) right--;
            while (left < right && a[left] <= keyValue) left++;
            if (left < right) {
                int temp = a[left];
                a[left] = a[right];
                a[right] = temp;
            }
        }
        a[low] = a[left];    // 此时必有left == right
        a[left] = keyValue;    // 轴元素放到了其该在的位置
        return left;
    }
}
```

在选定最左侧low为轴的前提下，若欲升序排列则应right指针先向左动，若欲降序排列则应left指针先向右动。其原因在于while结束后将进行哨兵(即left与right交汇的那个位置)与low的交换，若欲升序排列，则必须保证哨兵要不大于keyValue才行，而欲保证这一点，则只能在每次循环的内部，均先移动right，反之同理。

# 插入排序

## 直接插入排序

```
public class DirectInsertSort {

    public static void directInsertSort(int[] a) {
        if (null == a)
            throw new NullPointerException("a is null");
        for (int keyIndex = 1; keyIndex < a.length; keyIndex++) {
            int keyValue = a[keyIndex];
            int preIndex = keyIndex - 1;
            while (preIndex >= 0 && a[preIndex] > keyValue) {
                a[preIndex + 1] = a[preIndex];
                preIndex--;
            }
            a[preIndex + 1] = keyValue;
        }
    }
}
```

## 希尔排序

```
public class ShellSort {

    public static void shellSort(int[] a) {
        if (null == a)
            throw new NullPointerException("a is null");
        int base = 3;    // base > 1
        int gap = 1;
        // 必须保证最后一轮排序的步长=1
        while (gap < a.length / base) gap = base * gap + 1;
        while (gap > 0) {
            for (int keyIndex = gap; keyIndex < a.length; keyIndex++) {
                int keyValue = a[keyIndex];
                int preIndex = keyIndex - gap;
                while (preIndex >= 0 && a[preIndex] > keyValue) {
                    a[preIndex + gap] = a[preIndex];
                    preIndex -= gap;
                }
                a[preIndex + gap] = keyValue;
            }
            gap /= base;
        }
    }
}
```

# 选择排序

## 简单选择排序

```
public class SimpleChooseSort {

    public static void simpleChooseSort(int[] a) {
        if (null == a)
            throw new NullPointerException("a is null");
        for (int i = 0; i < a.length - 1; i++) {
            int minIndex = i;
            for (int j = i + 1; j < a.length; j++) {
                if (a[j] < a[minIndex]) minIndex = j;
            }
            if (minIndex != i) swap(a, i, minIndex);
        }
    }

    private static void swap(int[] a, int i, int j) {
        int temp = a[i];
        a[i] = a[j];
        a[j] = temp;
    }
}
```

## 堆排序

```
public class HeapSort {

    public static void heapSort(int[] a) {
        if (null == a)
            throw new NullPointerException("a is null");
        for (int i = getPI(a.length - 1); i >= 0; i--) createMaxHeap(a, a.length, i);
        for (int i = a.length - 1; i > 0; i--) {
            swap(a, 0, i);
            createMaxHeap(a, i, 0);
        }
    }

    /**
     * 创建大根堆
     * @param a
     * @param mri 创建大根堆的元素范围：i < mri
     * @param ri 需创建大根堆的根
     */
    private static void createMaxHeap(int[] a, int mri, int ri) {
        int lci = getLCI(ri);
        int rci = getRCI(ri);
        int li = ri;
        if (lci < mri && a[lci] > a[li]) li = lci;
        if (rci < mri && a[rci] > a[li]) li = rci;
        if (li != ri) {
            swap(a, ri, li);
            createMaxHeap(a, mri, li);
        }
    }

    private static void swap(int[] a, int i, int j) {
        int temp = a[i];
        a[i] = a[j];
        a[j] = temp;
    }

    private static int getPI(int i) {
        return (i - 1) / 2;
    }

    private static int getLCI(int i) {
        return 2 * i + 1;
    }

    private static int getRCI(int i) {
        return 2 * i + 2;
    }
}
```

# 归并排序

```
public class MergeSort {

    public static void mergeSort(int[] a) {
        if (null == a)
            throw new NullPointerException("a is null");
        mergeSort(a, new int[a.length], 0, a.length - 1);
    }

    private static void mergeSort(int[] a, int[] temp, int begin, int end) {
        if (begin >= end) return;
	int mid = begin + (end - begin) / 2;
        int b1 = begin, e1 = mid;
        int b2 = mid + 1, e2 = end;
        mergeSort(a, temp, b1, e1);    // 保证左面一半有序
        mergeSort(a, temp, b2, e2);    // 保证右面一半有序
        int i = begin;
        while (b1 <= e1 && b2 <= e2) temp[i++] = a[b1] <= a[b2] ? a[b1++] : a[b2++];
        while (b1 <= e1) temp[i++] = a[b1++];
        while (b2 <= e2) temp[i++] = a[b2++];
        for (i = begin; i <= end; i++) a[i] = temp[i];
    }
}
```