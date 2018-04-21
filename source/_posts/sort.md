title: 排序算法学习
date: 2016-03-29 09:49:55
tags: [技术]
---

### 插入排序

### 归并排序

### 快速排序

``` cpp
void qsort(int *arr, int start, int end) {
    if (arr == NULL || start >= end) {
        return;
    }
    int small = start;
    swap(arr[start], arr[end]);

    for (int i = start; i < end; ++i) {
        if (arr[i] < arr[end]) {
            if (i != small) {
                swap(arr[i], arr[small]);
            }
            small += 1;
        }
    }

    swap(arr[small], arr[end]);

    qsort(arr, start, small-1);
    qsort(arr, small+1, end);
}
```