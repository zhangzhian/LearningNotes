# Leetcode数据结构与算法

### [0001]求1+2+…+n

求 `1+2+...+n` ，要求不能使用乘除法、for、while、if、else、switch、case等关键字及条件判断语句（A?B:C）。

**示例 1：**

```
输入: n = 3
输出: 6
```

**示例 2：**

```
输入: n = 9
输出: 45
```

**限制：**

- `1 <= n <= 10000`

**解答：**

等差数列求和公式

```cpp
class Solution {
    
    public int sumNums(int n) {
        return (int) (Math.pow(n, 2) + n) >> 1;
    }
}
```

递归

```java
class Solution {

    public int sumNums(int n) {
        int sum = n;
        boolean flag = n > 0 && (sum += sumNums(n - 1)) > 0;
        return sum;
    }
}
```

### [0002]反转链表

反转一个单链表。

**示例**：

```
输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL
```

**进阶**：

你可以迭代或递归地反转链表。你能否用两种方法解决这道题？

**方法一：迭代**

假设存在链表 1 → 2 → 3 → Ø，我们想要把它改成 Ø ← 1 ← 2 ← 3。

在遍历列表时，将当前节点的 next 指针改为指向前一个元素。由于节点没有引用其上一个节点，因此必须事先存储其前一个元素。在更改引用之前，还需要另一个指针来存储下一个节点。不要忘记在最后返回新的头引用！

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
public ListNode reverseList(ListNode head) {
    //申请节点，pre和 cur，pre指向null
    ListNode pre = null;
    ListNode cur = head;
  	ListNode tmp = null;
    while (cur != null) {
        tmp = cur.next;//记录当前节点的下一个节点
        cur.next = pre;//然后将当前节点指向pre
        //pre和cur节点都前进一位
        pre = cur;
        cur = tmp;
    }
    return pre;
}
```

**复杂度分析**

- 时间复杂度：O(n)，假设 n*n* 是列表的长度，时间复杂度是 O(n)。
- 空间复杂度：O(1)。

**方法二：递归**

递归的两个条件：

1. 终止条件是当前节点或者下一个节点==null
2. 在函数内部，改变节点的指向，也就是 head 的下一个节点指向 head 递归函数那句

```java
head.next.next = head
```

很不好理解，其实就是 head 的下一个节点指向head。
递归函数中每次返回的 cur 其实只最后一个节点，在递归函数内部，改变的是当前节点的指向。

```java
public ListNode reverseList(ListNode head) {
    if (head == null || head.next == null) return head;
    ListNode p = reverseList(head.next);
    head.next.next = head;
    head.next = null;
    return p;
}
```

**复杂度分析**

- 时间复杂度：O(n)，假设 n 是列表的长度，那么时间复杂度为O(n)。

- 空间复杂度：O(n)，由于使用递归，将会使用隐式栈空间。递归深度可能会达到 n 层。

### [0003]两数之和

给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。

你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素。

示例:

```java
给定 nums = [2, 7, 11, 15], target = 9

因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]
```

**方法一：暴力法**

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        for (int i = 0; i < nums.length; i++) {
            for (int j = i + 1; j < nums.length; j++) {
                if (nums[j] == target - nums[i]) {
                    return new int[] { i, j };
                }
            }
        }
        throw new IllegalArgumentException("No two sum solution");
    }
}
```

时间复杂度：O(n2)

空间复杂度：O(1)

**方法二：两遍哈希表**

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> map = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            map.put(nums[i], i);
        }
        for (int i = 0; i < nums.length; i++) {
            int complement = target - nums[i];
            if (map.containsKey(complement) && map.get(complement) != i) {
                return new int[] { i, map.get(complement) };
            }
        }
        throw new IllegalArgumentException("No two sum solution");
    }
}
```

时间复杂度：O(n)

空间复杂度：O(n)

**方法三：一遍哈希表**

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> map = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            int complement = target - nums[i];
            if (map.n(complement)) {
                return new int[] {, i };
            }
            map.put(nums[i], i);
        }
        throw new IllegalArgumentException("No two sum solution");
    }
}
```

时间复杂度：O(n)

空间复杂度：O(n)

### [0004]合并排序的数组

给定两个排序后的数组 A 和 B，其中 A 的末端有足够的缓冲空间容纳 B。 编写一个方法，将 B 合并入 A 并排序。

初始化 A 和 B 的元素数量分别为 m 和 n。

示例:

```java
输入:
A = [1,2,3,0,0,0], m = 3
B = [2,5,6],       n = 3

输出: [1,2,2,3,5,6]
```

**方法 1: 直接合并后排序**

```java
class Solution {
    public void merge(int[] A, int m, int[] B, int n) {
        //System.arraycopy(B, 0 , A, m, n);
        for (int i = 0; i != n; ++i)
            A[m + i] = B[i];       
        Arrays.sort(A);
    }
}
```

时间复杂度：O((m+n)log⁡(m+n))

空间复杂度：O(log⁡(m+n))

**方法 2: 双指针**

```java
class Solution {
    public void merge(int[] A, int m, int[] B, int n) {
        int pa = 0, pb = 0;
        int sorted[] = new int[m + n];
        int cur;
        while (pa < m || pb < n) {
            if (pa == m)
                cur = B[pb++];
            else if (pb == n)
                cur = A[pa++];
            else if (A[pa] < B[pb])
                cur = A[pa++];
            else
                cur = B[pb++];
            sorted[pa + pb - 1] = cur;
        }
        for (int i = 0; i != m + n; ++i)
            A[i] = sorted[i];
    }
}
```

时间复杂度：O(m+n)

空间复杂度：O(m+n)

**方法3：逆向双指针**

```java
class Solution {
    public void merge(int[] A, int m, int[] B, int n) {
        // 将其中一个数组中的数字遍历完
        while (m > 0 && n > 0) {
            // 对比选出较大的数放在 m + n - 1 的位置，并将选出此数的指针向前移动
            A[m + n - 1] = A[m - 1] > B[n - 1] ? A[m-- - 1] : B[n-- - 1];
        }
        // 剩下的数都比已经遍历过的数小
        // 如果 m 不为 0，则 A 没遍历完，都已经在 A 中不用再管
        // 如果 n 不为 0，则 B 没遍历完，直接全移到 A 中相同的位置
        while (n > 0) {
            A[n - 1] = B[n - 1];
            n--;
        }
    }
}
```

时间复杂度：O(m+n)

空间复杂度：O(1)

### [0005]分糖果 II

排排坐，分糖果。

我们买了一些糖果 candies，打算把它们分给排好队的 n = num_people 个小朋友。

给第一个小朋友 1 颗糖果，第二个小朋友 2 颗，依此类推，直到给最后一个小朋友 n 颗糖果。

然后，我们再回到队伍的起点，给第一个小朋友 n + 1 颗糖果，第二个小朋友 n + 2 颗，依此类推，直到给最后一个小朋友 2 * n 颗糖果。

重复上述过程（每次都比上一次多给出一颗糖果，当到达队伍终点后再次从队伍起点开始），直到我们分完所有的糖果。注意，就算我们手中的剩下糖果数不够（不比前一次发出的糖果多），这些糖果也会全部发给当前的小朋友。

返回一个长度为 num_people、元素之和为 candies 的数组，以表示糖果的最终分发情况（即 ans[i] 表示第 i 个小朋友分到的糖果数）。

**示例 1：**

```
输入：candies = 7, num_people = 4
输出：[1,2,3,1]
解释：
第一次，ans[0] += 1，数组变为 [1,0,0,0]。
第二次，ans[1] += 2，数组变为 [1,2,0,0]。
第三次，ans[2] += 3，数组变为 [1,2,3,0]。
第四次，ans[3] += 1（因为此时只剩下 1 颗糖果），最终数组变为 [1,2,3,1]。
```

**示例 2：**

```
输入：candies = 10, num_people = 3
输出：[5,2,3]
解释：
第一次，ans[0] += 1，数组变为 [1,0,0]。
第二次，ans[1] += 2，数组变为 [1,2,0]。
第三次，ans[2] += 3，数组变为 [1,2,3]。
第四次，ans[0] += 4，最终数组变为 [5,2,3]。
```

**提示：**

```
1 <= candies <= 10^9
1 <= num_people <= 1000
```

方法 1: 暴力

思路

最直观的方法是不断地遍历数组，如果还有糖就一直分，直到没有糖为止。

```java
class Solution {
    public int[] distributeCandies(int candies, int num_people) {
        int[] ans = new int[num_people];
        int i = 0;
        while (candies != 0) {
            ans[i % num_people] += Math.min(candies, i + 1);
            candies -= Math.min(candies, i + 1);
            i += 1;
        }
        return ans;
    }
}
```

复杂度

![复杂度](https://github.com/zhangzhian/LearningNotes/blob/master/res/05复杂度.png?raw=true)

方法 2：等差数列求和

思路

这是一个数学问题，可以对其简化。

更好的做法是使用一个简单的公式代表糖果分配，可以在 O(N) 时间内完成糖果分发，并生成最终的分配数组。

逐步推导该公式:

https://leetcode-cn.com/problems/distribute-candies-to-people/solution/fen-tang-guo-ii-by-leetcode-solution/

https://leetcode-cn.com/problems/distribute-candies-to-people/solution/xiang-xi-jie-shi-shu-xue-fang-fa-zen-yao-zuo-gao-z/

```java
class Solution {
  public int[] distributeCandies(int candies, int num_people) {
    int n = num_people;
    // how many people received complete gifts
    int p = (int)(Math.sqrt(2 * candies + 0.25) - 0.5);
    int remaining = (int)(candies - (p + 1) * p * 0.5);
    int rows = p / n, cols = p % n;

    int[] d = new int[n];
    for(int i = 0; i < n; ++i) {
      // complete rows
      d[i] = (i + 1) * rows + (int)(rows * (rows - 1) * 0.5) * n;
      // cols in the last row
      if (i < cols) d[i] += i + 1 + rows * n;
    }
    // remaining candies        
    d[cols] += remaining;
    return d;
  }
}
```

复杂度分析

时间复杂度：O(N)，计算 N 个人的糖果数量。

空间复杂度：O(1)，除了答案数组只需要常数空间来存储若干变量。

### [0006]有多少小于当前数字的数字

给你一个数组 nums，对于其中每个元素 nums[i]，请你统计数组中比它小的所有数字的数目。

换而言之，对于每个 nums[i] 你必须计算出有效的 j 的数量，其中 j 满足 j != i 且 nums[j] < nums[i] 。

以数组形式返回答案。

示例 1：

```
输入：nums = [8,1,2,2,3]
输出：[4,0,1,1,3]
```

解释： 

对于 nums[0]=8 存在四个比它小的数字：（1，2，2 和 3）。 

对于 nums[1]=1 不存在比它小的数字。

对于 nums[2]=2 存在一个比它小的数字：（1）。 

对于 nums[3]=2 存在一个比它小的数字：（1）。 

对于 nums[4]=3 存在三个比它小的数字：（1，2 和 2）。

示例 2：

```
输入：nums = [6,5,4,8]
输出：[2,1,0,3]
```

示例 3：

```
输入：nums = [7,7,7,7]
输出：[0,0,0,0]
```

提示：

```
2 <= nums.length <= 500
0 <= nums[i] <= 100
```

方法一：暴力

```java
class Solution {
    public int[] smallerNumbersThanCurrent(int[] nums) {
        int[] res = new int[nums.length];
        for(int i = 0;i < nums.length; i++)
            for(int j = 0;j < nums.length; j++)
                if(nums[i] > nums[j])
                    res[i]++;
        return res;
    }
}
```

复杂度分析

时间复杂度：枚举数组里的每个数字为 O(n) ，遍历数组也为 O(n)，所以总时间复杂度为两者相乘，即 O(n2) ，其中 n=nums.length 。

空间复杂度：O(1) ，不需要使用额外的空间。

方法二：频次数组 + 前缀和

https://leetcode-cn.com/problems/how-many-numbers-are-smaller-than-the-current-number/solution/you-duo-shao-xiao-yu-dang-qian-shu-zi-de-shu-zi--2/

```java
class Solution {
    public int[] smallerNumbersThanCurrent(int[] nums) {
        int[] res = new int[nums.length];
        int[] temp = new int[101];
        for(int i = 0;i < nums.length; i++)
            temp[nums[i]]++;
        for(int i = 1;i < temp.length; i++)
            temp[i] += temp[i-1]; // 求前缀和
        for(int i = 0;i < nums.length; i++)
            if (nums[i]!=0) 
                res[i] = temp[nums[i] - 1];
        return res;
    }
}
```

时间复杂度：O(S+n) ，其中 S 为值域大小，n=nums.length 。

空间复杂度：O(S) ，需要开一个值域大小的数组。

**方法三：排序**

```java
public int[] smallerNumbersThanCurrent(int[] nums) { // 8, 1, 2, 2, 3
    int len = nums.length;
    Map<Integer, Set<Integer>> valueIndex = new HashMap<>(len); // 预存每个值与索引对应
    for (int i = 0; i < len; i++) {
        if (!valueIndex.containsKey(nums[i])) valueIndex.put(nums[i], new HashSet<>());
        valueIndex.get(nums[i]).add(i);
    }
    int[] sortedArr = Arrays.copyOf(nums, len);
    int[] res = new int[len];
    Arrays.sort(sortedArr); // 1, 2, 2, 3, 8
    for (int si = len - 1; si >= 0; si--) {
        for (int i : valueIndex.get(sortedArr[si])) res[i] = si; // 同值的所有索引都更新
    }
    return res;
}
```

时间复杂度 O(nlog(n))，空间复杂度 O(n)



