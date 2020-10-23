# 旋转矩阵(rotate-matrix-lcci)

给你一幅由 N × N 矩阵表示的图像，其中每个像素的大小为 4 字节。请你设计一种算法，将图像旋转 90 度。

不占用额外内存空间能否做到？

 

示例 1:

```
给定 matrix = 
[
  [1,2,3],
  [4,5,6],
  [7,8,9]
],

原地旋转输入矩阵，使其变为:
[
  [7,4,1],
  [8,5,2],
  [9,6,3]
]
```

示例 2:

```
给定 matrix =
[
  [ 5, 1, 9,11],
  [ 2, 4, 8,10],
  [13, 3, 6, 7],
  [15,14,12,16]
], 

原地旋转输入矩阵，使其变为:
[
  [15,13, 2, 5],
  [14, 3, 4, 1],
  [12, 6, 8, 9],
  [16, 7,10,11]
]
```



# 思路和代码

本题解参考[官方](https://leetcode-cn.com/problems/rotate-matrix-lcci/solution/xuan-zhuan-ju-zhen-by-leetcode-solution/)。

## 观察矩阵的第一行，它旋转后的变化

```
 5  1  9 11              x  x  x  5
 x  x  x  x   =旋转后=>   x  x  x  1
 x  x  x  x              x  x  x  9
 x  x  x  x              x  x  x 11
```

看一下**坐标变化**情况

```
5--->	(0,0)	 旋转=>	(0,3)
1--->	(0,1)	 旋转=>	(1,3)
9--->	(0,2)	 旋转=>	(2,3)
11-->	(0,3)	 旋转=>	(3,3)
```

我们可以直接看出来的规律是

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200407140227.jpg)

原坐标的列，变为了新坐标的行

那新坐标的列是怎么变化的呢？

一个栗子有可能看不出来，再看一个：

## 观察矩阵的第二行，它旋转后的变化

```
 x  x  x  x              x  x  2  x
 2  4  8 10   =旋转后=>   x  x  4  x
 x  x  x  x              x  x  8  x
 x  x  x  x              x  x 10  x
```

看一下**坐标变化**情况

```
2--->	(1,0)	 旋转=>	(0,2)
4--->	(1,1)	 旋转=>	(1,2)
8--->	(1,2)	 旋转=>	(2,2)
10-->	(1,3)	 旋转=>	(3,2)
```

原坐标的列，变为了新坐标的行

那新坐标的列是怎么变化的呢？

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200407141406.jpg)



发现什么没有

```
0+3=3
1+2=3
```

两个栗子有可能看不出来，再看一个：

## 观察矩阵的第三行，它旋转后的变化

```
 x  x  x  x              x  13 x  x
 x  x  x  x   =旋转后=>   x  3  x  x
 13 3  6  7              x  6  x  x
 x  x  x  x              x  7  x  x
```

看一下**坐标变化**情况

```
13-->	(2,0)	 旋转=>	(0,1)
3--->	(2,1)	 旋转=>	(1,1)
6--->	(2,2)	 旋转=>	(2,1)
7--->	(2,3)	 旋转=>	(3,1)
```

发现什么没有

```
0+3=3
1+2=3
2+1=3
```

因为数组下标是从0开始的，所以

`old_row + new_col = N - 1`

把`old_row`右移，得

`new_col = N - old_row - 1`

所以，变化情况是

`matrix[row][col] ` 旋转后变为`matrix[col][n-row-1]`。



这样以来，我们使用一个与matrix大小相同的辅助数组matrix_new,临时存储旋转后的结果。

我们遍历matrix 中的每一个元素，根据上述规则将该元素存放到matrix_new中对应的位置。

在遍历完成之后，再将 matrix_new中的结果复制到原数组中即可。




# 代码


```c++
class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        int n = matrix.size();
        // C++ 这里的 = 拷贝是值拷贝，会得到一个新的数组
        auto matrix_new = matrix;
        for (int i = 0; i < n; ++i) {
            for (int j = 0; j < n; ++j) {
                matrix_new[j][n - i - 1] = matrix[i][j];
            }
        }
        // 这里也是值拷贝
        matrix = matrix_new;
    }
};
```

