# Leetcode数据结构与算法

### [0017] 二进制链表转整数

给你一个单链表的引用结点 head。链表中每个结点的值不是 0 就是 1。已知此链表是一个整数数字的二进制表示形式。

请你返回该链表所表示数字的 十进制值 。

示例 1：

```
输入：head = [1,0,1]
输出：5
解释：二进制数 (101) 转化为十进制数 (5)
```


示例 2：

```
输入：head = [0]
输出：0
```


示例 3：

```
输入：head = [1]
输出：1
```


示例 4：

```
输入：head = [1,0,0,1,0,0,1,1,1,0,0,0,0,0,0]
输出：18880
```


示例 5：

```
输入：head = [0,0]
输出：0
```


提示：

链表不为空。
链表的结点总数不超过 30。
每个结点的值不是 0 就是 1。



方法一：

```java
 class Solution {
        public int getDecimalValue(ListNode head) {
            int sum = 0;
            while(head!=null){
                sum = sum*2 + head.val;
                head = head.next;
            }
            return sum;
        }
    }
```

  方法二：

```java
class Solution {
    public int getDecimalValue(ListNode head) {
        int sum = 0;
         while(head!=null){
            sum = (sum << 1) | (head.val);
            head = head.next;
        }
        return sum;
    }
}
```

- 时间复杂度O(N)

- 空间复杂度O(1)

### [0018]访问所有点的最小时间

平面上有 n 个点，点的位置用整数坐标表示 points[i] = [xi, yi]。请你计算访问所有这些点需要的最小时间（以秒为单位）。

你可以按照下面的规则在平面上移动：

- 每一秒沿水平或者竖直方向移动一个单位长度，或者跨过对角线（可以看作在一秒内向水平和竖直方向各移动一个单位长度）。
- 必须按照数组中出现的顺序来访问这些点。

**示例 1：**

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2019/11/24/1626_example_1.png)`

```
输入：points = [[1,1],[3,4],[-1,0]]
输出：7
解释：一条最佳的访问路径是： [1,1] -> [2,2] -> [3,3] -> [3,4] -> [2,3] -> [1,2] -> [0,1] -> [-1,0]   
从 [1,1] 到 [3,4] 需要 3 秒 
从 [3,4] 到 [-1,0] 需要 4 秒
一共需要 7 秒
```

方法一：切比雪夫距离

对于平面上的两个点 x = (x0, x1) 和 y = (y0, y1)，设它们横坐标距离之差为 dx = |x0 - y0|，纵坐标距离之差为 dy = |x1 - y1|，对于以下三种情况，我们可以分别计算出从 x 移动到 y 的最少次数：

- dx < dy：沿对角线移动 dx 次，再竖直移动 dy - dx 次，总计 dx + (dy - dx) = dy 次；

- dx == dy：沿对角线移动 dx 次；

- dx > dy：沿对角线移动 dy 次，再水平移动 dx - dy 次，总计 dy + (dx - dy) = dx 次。

可以发现，对于任意一种情况，从 x 移动到 y 的最少次数为 dx 和 dy 中的较大值 max(dx, dy)，这也被称作 x 和 y 之间的 切比雪夫距离。

由于题目要求，需要按照数组中出现的顺序来访问这些点。因此我们遍历整个数组，对于数组中的相邻两个点，计算出它们的切比雪夫距离，所有的距离之和即为答案。

```java
class Solution {
    public int minTimeToVisitAllPoints(int[][] points) {
        int x0 = points[0][0];
        int x1 = points[0][1];
        int ans = 0;
        for(int i = 1; i < points.length; ++i){
            int y0 = points[i][0];
            int y1 = points[i][1];
            ans += Math.max(Math.abs(x0 - y0), Math.abs(x1 - y1));
            x0 = y0;
            x1 = y1;
        }
        return ans;
    }
}
```

**复杂度分析**

- 时间复杂度：O(N)
- 空间复杂度：O(1)

### [0019]删除链表中的节点

请编写一个函数，使其可以删除某个链表中给定的（非末尾）节点，你将只被给定要求被删除的节点。

现有一个链表 -- head = [4,5,1,9]，它可以表示为:

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2019/01/19/237_example.png)

示例 1:

```
输入: head = [4,5,1,9], node = 5
输出: [4,1,9]
解释: 给定你链表中值为 5 的第二个节点，那么在调用了你的函数之后，该链表应变为 4 -> 1 -> 9.
```

示例 2:

```
输入: head = [4,5,1,9], node = 1
输出: [4,5,9]
解释: 给定你链表中值为 1 的第三个节点，那么在调用了你的函数之后，该链表应变为 4 -> 5 -> 9.
```

说明:

```
链表至少包含两个节点。
链表中所有节点的值都是唯一的。
给定的节点为非末尾节点并且一定是链表中的一个有效节点。
不要从你的函数中返回任何结果。
```

方法一：

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public void deleteNode(ListNode node) {
        node.val = node.next.val;
        node.next = node.next.next;
    }
}
```

时间和空间复杂度都是：O(1)

###[0020]统计有序矩阵中的负数

给你一个 `m * n` 的矩阵 `grid`，矩阵中的元素无论是按行还是按列，都以非递增顺序排列。 

请你统计并返回 `grid` 中 **负数** 的数目。

示例 1：

```
输入：grid = [[4,3,2,-1],[3,2,1,-1],[1,1,-1,-2],[-1,-1,-2,-3]]
输出：8
解释：矩阵中共有 8 个负数。
```

示例 2：

```
输入：grid = [[3,2],[1,0]]
输出：0
```

示例 3：

```
输入：grid = [[1,-1],[-1,-1]]
输出：3
```

示例 4：

```
输入：grid = [[-1]]
输出：1
```


提示：

```
m == grid.length
n == grid[i].length
1 <= m, n <= 100
-100 <= grid[i][j] <= 100
```

方法一：暴力

```java
class Solution {
    public int countNegatives(int[][] grid) {
        int m = grid.length; //row
        int n = grid[0].length;//col
        int r = 0;
        for(int i=0;i<m;i++){
            for(int j=0;j<n;j++){
                if(grid[i][j] < 0)
                    r++;
            }
        }
        return r;
    }
}
```

```java
class Solution {
    public int countNegatives(int[][] grid) {
        int m = grid.length; //row
        int n = grid[0].length;//col
        int r = 0;
        for(int i=0;i<m;i++){
            for(int j=0;j<n;j++){
                if(grid[i][j] < 0){
                    r += n-j;
                    break;
                }          
            }
        }
        return r;
    }
}
```

- 时间复杂度：O(nm)
- 空间复杂度：O(1)

方法二：二分查找

```java
class Solution {
    public int countNegatives(int[][] grid) {
        int count = 0, m = grid.length, n = grid[0].length;
        for (int i = 0; i < m; i++) {
            int[] row = grid[i];
            if (row[n - 1] >= 0) continue; // 整行非负，跳过
            if (row[0] < 0) { // 整行负数
                count += (m - i) * n; // 后面的行也计入
                break; // 无需再继续遍历
            }
            int first = _binarySearch(row); // 当前行二分查找第一个小于 0 的数的索引
            count += n - first;
        }
        return count;
    }

    // 查找第一个小于 0 的数的索引
    private int _binarySearch(int[] arr) {
        int begin = 0, end = arr.length;
        while (begin < end) {
            int mid = begin + ((end - begin) >> 1);
            if (arr[mid] >= 0) begin = mid + 1;
            else { // 负数之后，还要再判断前一个不是负数
                if (arr[mid - 1] >= 0) return mid;
                end = mid;
            }
        }
        return begin;
    }
}
```

- 时间复杂度：二分查找一行的时间复杂度为logm，需要遍历n行，所以总时间复杂度是O(nlogm)。
- 空间复杂度：O(1)。

方法三：分治

https://leetcode-cn.com/problems/count-negative-numbers-in-a-sorted-matrix/solution/tong-ji-you-xu-ju-zhen-zhong-de-fu-shu-by-leetcode/

- 时间复杂度：T*(*n*)=2*T*(*n*/2)+*O*(*n*)=*O*(*nlogn)

- 空间复杂度：O(1)

方法四：倒序遍历

```java
class Solution {
    public int countNegatives(int[][] grid) {
        int m = grid[0].length;
        int pos = grid[0].length -1;
        int num=0;
        for(int i = 0;i < grid.length ;i++){
            int j = 0;
            for(j=pos;j>=0;--j){
                if(grid[i][j] >= 0){
                    if(j+1 < m){
                        pos = j+1;
                        num += m-pos;
                    }
                    break;
                } 
            }
            if(j == -1){
                num += m;
                pos = -1;
            }
        }
        return num;
    }
}
```

时间复杂度：考虑每次循环变量的起始位置是单调不降的，所以起始位置最多移动m 次，时间复杂度 O(m)。
空间复杂度：O(1)

