title: 寻找前K大的数
date: 2014-12-02 18:51:06
tags: [技术]
---
找出N个整数里面前K大的整数


思路1: 使用选择或冒泡算法，排出前K个元素，时间复杂度为O(N*K)

思路2: 对这N个数排序，输出最大的K个，时间复杂度O(N*log(N))

思路3: 快速排序的变种。前面寻找数组中第K大数的过程中，当找准数组中第K大数的位置时，数组中比K大的数据都在K的左边，比K小的数据都在K的右边。从而获取前K大的数据。其实也是部分排序。算法复杂度:O(N)

思路4: 将前面K个元素构建为最小堆，将后面N - K个元素一次与堆顶比较，如果比堆顶元素大，则与堆顶交换，并将前面K个元素调整为最小堆，时间复杂度为O(N*log(K))
###附最小堆实现代码
```cpp
#include <algorithm>
#include <cstdio>
using namespace std;
// H是最小堆, heap_size是当前堆的大小
int H[10000], heap_size;
// 递归维护最小堆
void MIN_HEADPIFY(int i) {
    int l = 2 * i, r = 2 * i + 1, largest = i;
    if (l <= heap_size && H[l] < H[i]) {
        largest = l;
    }
    if (r <= heap_size && H[r] < H[largest]) {
        largest = r;
    }
    if (largest != i) {
        swap(H[i], H[largest]);
        MIN_HEADPIFY(largest);
    }
}
int main() {
    int n, k, tmp;
    while (~scanf("%d%d", &n, &k)) {
        heap_size = k;
        for (int i = 0; i < k; i++) {
            scanf("%d", &H[i + 1]);
        }
        // 对前k个数建最小堆
        for (int i = k / 2; i >= 1; i--) {
            MIN_HEADPIFY(i);
        }
        for (int i = 0; i < n - k; i++) {
            scanf("%d", &tmp);
            // 如果读入剩余的数比堆顶大, 则插入堆顶
            // 并对堆顶进行维护
            if (tmp > H[1]) {
                H[1] = tmp;
                MIN_HEADPIFY(1);
            }
        }
        // 得到的H[1]---H[k]即为前k大数的集合
        // 对其进行排序即可得到前k大的数
        sort(H + 1, H + k + 1);
        for (int i = k; i > 1; i--) {
            printf("%d ", H[i]);
        }
        printf("%d\n", H[1]);
    }
    return 0;
}
```
