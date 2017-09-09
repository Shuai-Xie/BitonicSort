[双调排序介绍 - 简书](http://www.jianshu.com/p/5f082115ac27)

[分段双调排序 - 简书](http://www.jianshu.com/p/def12036394d)


## 分段双调排序
给出分成 m 段的 n 个浮点数，输入数据已按段号有序，但每段内部无序。
用 C/C++ 编写一个分段双调排序(Bitonic sort)函数，对每一段内部的浮点数进行排序，但不要改变段间的位置。 

### 1. 接口方式
```C++
// 分段双调排序
void segmentedBitonicSort(float* data, int* seg_id, int* seg_start, int n, int m); 
```
- data 包含需要分段排序的 n 个 float 值**（由下面的 seg_id 可知是表示多段的数据）**
- seg_id **给出 data 中 n 个元素各自所在的段编号**
- seg_start 共有 m+1 个元素，**前 m 个分别给出 0..m-1 共 m 个段的起始位置，seg_start[m] = n**
- n 表示 data 中包含 n 个数据
- m 表示 data 分为 m 段数据

由题意得，m <= n，因为 data 某一段可能有多于 1 个数据，这种情况下：m < n

seg_id 中的元素保证**单调不下降**，即对任意的 i < j，seg_id[i] <= seg_id[j]
seg_id 所有元素均在 0 到 m-1 范围内。（因为是段 id，m 段就是0..m-1） 

### 2. 输入输出
输入
```C++
float data[5] = {0.8, 0.2, 0.4, 0.6, 0.5};
int seg_id[5] = {0, 0, 1, 1, 1}; // 0,1 所在位置元素对应的段id
int seg_start[3] = {0, 2, 5}; // 表示第1段从0开始:{0.8, 0.2}，第2段从2开始:{0.4, 0.6, 0.5}
int n = 5; // 总共5个数
int m = 2; // 2段
```

输出
```C++
float data[5] = {0.2, 0.8, 0.4, 0.5, 0.6};
```
### 3. 其他要求

#### 3.1 注意
1. 必须使用双调排序算法进行排序。 
2. 可以直接使用从网上下载的双调排序代码，但须注明出处。 

#### 3.2 加分挑战（非必需）
1. 不递归：segmentedBitonicSort 函数及其所调用的任何其他函数都不得直接或间接地进行递归。 
1. 不调用函数：segmentedBitonicSort 不调用除标准库函数外的任何其他函数。 
1. 内存高效：segmentedBitonicSort 及其所调用的任何其他函数都不得进行动态内存分配，包括malloc、new和静态定义的STL容器。 
1. 可并行：segmentedBitonicSort 涉及到的所有时间复杂度O(n)以上的代码都写在for循环中，而且每个这样的for循环内部的循环顺序可以任意改变，不影响程序结果。注：自己测试时可以用rand()决定循环顺序。 
1. 不需内存：segmentedBitonicSort 不调用任何函数（包括C/C++标准库函数）， 不使用全局变量，所有局部变量都是int、float或指针类型，C++程序不使用new关键字。 
1. 绝对鲁棒：在输入数据中包含 NaN 时（例如```sqrt(-1.0)```)，保证除NaN以外的数据正确排序，NaN的个数保持不变。 

>NaN，是 Not a Number 的缩写。NaN 用于处理计算中出现的错误情况，比如 0.0 除以 0.0 或者求负数的平方根。

你的程序每满足以上的一个条件都可以获得额外的加分。 

#### 3.3 应提交的结果

1. 算法描述； 
1. 尝试过和完成了的加分挑战； 
1. 可以独立运行的源代码； 
1. 测试数据； 
1. 性能分析； 
1. 测试的起始和完成时间以及实际使用的时间。 

#### 3.4 提示

1. 利用好网上资源。 
2. 尽量利用输入中的冗余信息。 
3. 利用好位操作。 （>>1）

***
## 解答
### 1. 递归分段双调排序
```C++
#include <iostream>
#include <iomanip>

using namespace std;

/**
 * @param arr 小数数列
 * @param len 数列长度
 * @param w 输出单个小数的长度
 * @param precision 小数的精度
 */
void printFloatArr(float *arr, int len) {
    for (int i = 0; i < len; ++i) {
        cout << setw(4) << arr[i];
    }
    cout << endl;
}

int getGreatest2nLessThan(int len) {
    int k = 1;
    while (k < len) k = k << 1; // 注意一定要加k=
    return k >> 1;
}

// 任意双调排序归并
void bitonicMergeAnyN(float *arr, int len, bool asd = true) {
    if (len > 1) {
        int m = getGreatest2nLessThan(len);
        for (int i = 0; i < len - m; ++i) {
            if (arr[i] > arr[i + m] == asd)
                swap(arr[i], arr[i + m]); // 根据asd判断是否交换
        }
        bitonicMergeAnyN(arr, m, asd); // 一般情况下，m > len-m
        bitonicMergeAnyN(arr + m, len - m, asd);
    }
}

// 双调排序
void bitonicSort(float *arr, int len, bool asd) { // asd 升序
    if (len > 1) {
        int m = len / 2;
        bitonicSort(arr, m, !asd); // 前半降序
        bitonicSort(arr + m, len - m, asd); // 后半升序
        bitonicMergeAnyN(arr, len, asd);
    }
}

/**
 * 分段双调排序
 * @param data 原始数据
 * @param seg_id data 中 n 个元素各自所在的段编号
 * @param seg_start 共有 m+1 个元素，前 m 个分别给出 0..m-1 共 m 个段的起始位置，seg_start[m] = n
 * @param n data 中包含 n 个数据
 * @param m data 分为 m 段数据
 */
void segmentedBitonicSort(float *data, int *seg_id, int *seg_start, int n, int m) {
    bool asd = true;
    for (int i = 0; i < m; ++i) {
        float *arr = data + seg_start[i]; // 数列起点
        int len = seg_start[i + 1] - seg_start[i]; // 数列长度
        bitonicSort(arr, len, asd);
        cout << "段 " << i << ": ";
        printFloatArr(arr, len);
    }
}


int main() {
    float data[7] = {0.8, 0.2, 0.4, 0.6, 0.5, 0.2, 0.1}; // 数列
    int seg_id[7] = {0, 0, 1, 1, 1, 2, 2}; // id
    int seg_start[4] = {0, 2, 5, 7}; // 段首位置
    int n = 7; // 数据长度
    int m = 3; // 数据段数

    segmentedBitonicSort(data, seg_id, seg_start, n, m);
}
```
```
段 0:  0.2 0.8
段 1:  0.4 0.5 0.6
段 2:  0.1 0.2
```

### 2. 解决NaN问题
在归并的时候，遇到NaN型数据，就直接跳到下一个数据。
```C++
// 任意双调排序归并
void bitonicMergeAnyN(float *arr, int len, bool asd = true) {
    if (len > 1) {
        int m = getGreatest2nLessThan(len);
        for (int i = 0; i < len - m; ++i) {

            // 解决NaN问题
            int low = i;
            while (arr[low] == NaN) low++;
            int high = i + m;
            while (arr[high] == NaN) high++;
            // 核心代码

            if (arr[low] > arr[high] == asd)
                swap(arr[low], arr[high]); // 根据asd判断是否交换
        }
        bitonicMergeAnyN(arr, m, asd); // 一般情况下，m > len-m
        bitonicMergeAnyN(arr + m, len - m, asd);
    }
}
```
完整代码
```C++
#include <iostream>
#include <iomanip>
#include <cmath>

#define NaN (float)sqrt(-1.0)

using namespace std;

/**
 * @param arr 小数数列
 * @param len 数列长度
 * @param w 输出单个小数的长度
 * @param precision 小数的精度
 */
void printFloatArr(float *arr, int len) {
    for (int i = 0; i < len; ++i) {
        if (arr[i] == NaN) cout << setw(5) << "NaN";
        else cout << setw(5) << arr[i];
    }
    cout << endl;
}

int getGreatest2nLessThan(int len) {
    int k = 1;
    while (k < len) k = k << 1; // 注意一定要加k=
    return k >> 1;
}

// 任意双调排序归并
void bitonicMergeAnyN(float *arr, int len, bool asd = true) {
    if (len > 1) {
        int m = getGreatest2nLessThan(len);
        for (int i = 0; i < len - m; ++i) {

            // 解决NaN问题
            int low = i;
            while (arr[low] == NaN) low++;
            int high = i + m;
            while (arr[high] == NaN) high++;

            if (arr[low] > arr[high] == asd)
                swap(arr[low], arr[high]); // 根据asd判断是否交换
        }
        bitonicMergeAnyN(arr, m, asd); // 一般情况下，m > len-m
        bitonicMergeAnyN(arr + m, len - m, asd);
    }
}

// 双调排序
void bitonicSort(float *arr, int len, bool asd) { // asd 升序
    if (len > 1) {
        int m = len / 2;
        bitonicSort(arr, m, !asd); // 前半降序
        bitonicSort(arr + m, len - m, asd); // 后半升序
        bitonicMergeAnyN(arr, len, asd);
    }
}

/**
 * 分段双调排序
 * @param data 原始数据
 * @param seg_id data 中 n 个元素各自所在的段编号
 * @param seg_start 共有 m+1 个元素，前 m 个分别给出 0..m-1 共 m 个段的起始位置，seg_start[m] = n
 * @param n data 中包含 n 个数据
 * @param m data 分为 m 段数据
 */
void segmentedBitonicSort(float *data, int *seg_id, int *seg_start, int n, int m) {
    bool asd = true;
    for (int i = 0; i < m; ++i) {
        float *arr = data + seg_start[i];
        int len = seg_start[i + 1] - seg_start[i];
        bitonicSort(arr, len, asd);
        cout << "段 " << i << ": ";
        printFloatArr(arr, len);
    }
}


int main() {

    float data[11] = {0.8, 0.2, 0.4, NaN, 0.6, 0.5, NaN, 0.2, 0.1, NaN, 0.01}; // 数列
    int seg_id[11] = {0, 0, 1, 1, 1, 1, 2, 2, 2, 2, 2}; // id
    int seg_start[4] = {0, 2, 6, 11}; // 段首位置
    int n = 11; // 数据长度
    int m = 3; // 数据段数

    segmentedBitonicSort(data, seg_id, seg_start, n, m);
}
```
```
段 0:   0.2  0.8
段 1:   0.4  NaN  0.5  0.6
段 2:   NaN 0.01  0.1  NaN  0.2
```
