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

### [0007]和为s的连续正数序列

输入一个正整数 target ，输出所有和为 target 的连续正整数序列（至少含有两个数）。

序列内的数字由小到大排列，不同序列按照首个数字从小到大排列。

https://leetcode-cn.com/problems/he-wei-sde-lian-xu-zheng-shu-xu-lie-lcof/solution/mian-shi-ti-57-ii-he-wei-sde-lian-xu-zheng-shu-x-2/

示例 1：

```
输入：target = 9
输出：[[2,3,4],[4,5]]
```


示例 2：

```
输入：target = 15
输出：[[1,2,3,4,5],[4,5,6],[7,8]]
```

限制：

```
1 <= target <= 10^5
```

**方法一：枚举 + 暴力**

```java
class Solution {
    public int[][] findContinuousSequence(int target) {
        List<int[]> list = new ArrayList<>();      
        // (target - 1) / 2 等效于 target / 2 下取整
        int sum = 0, limit = (target - 1) / 2; 
        for (int i = 1; i <= limit; ++i) {
            for (int j = i;; ++j) {
                sum += j;
                if (sum > target) {
                    sum = 0;
                    break;
                }
                else if (sum == target) {
                    int[] arr = new int[j-i+1];
                    for (int k = i; k <= j; k++) {
                        arr[k-i] = k;
                    }
                    list.add(arr);
                    sum = 0;
                    break;
                }
            }
        }
        return list.toArray(new int[list.size()][]);
    }
}
```

**方法二：枚举 + 数学优化**

```java
class Solution {
    public int[][] findContinuousSequence(int target) {
        List<int[]> list = new ArrayList<>();  
        // (target - 1) / 2 等效于 target / 2 下取整
        int limit = (target - 1) / 2; 
        for (int x = 1; x <= limit; ++x) {
            long delta = 1 - 4 * (x - 1L * x * x - 2 * target);
            if (delta < 0) continue;
            int delta_sqrt = (int)Math.sqrt(delta + 0.5);
            if (1L * delta_sqrt * delta_sqrt == delta 
            && (delta_sqrt - 1) % 2 == 0){
                // 另一个解(-1-delta_sqrt)/2必然小于0，不用考虑
                int y = (-1 + delta_sqrt) / 2; 
                if (x < y) {
                    int[] arr = new int[y-x+1];
                    for (int i = x; i <= y; i++){
                        arr[i-x] = i; 
                    } 
                    list.add(arr);
                }
            }
        }
        return list.toArray(new int[list.size()][]);
    }
}
```



**方法三：双指针（滑动窗口）**

https://leetcode-cn.com/problems/he-wei-sde-lian-xu-zheng-shu-xu-lie-lcof/solution/shi-yao-shi-hua-dong-chuang-kou-yi-ji-ru-he-yong-h/

 ```java
public int[][] findContinuousSequence(int target) {
    int i = 1; // 滑动窗口的左边界
    int j = 1; // 滑动窗口的右边界
    int sum = 0; // 滑动窗口中数字的和
    List<int[]> res = new ArrayList<>();

    while (i <= target / 2) {
        if (sum < target) {
            // 右边界向右移动
            sum += j;
            j++;
        } else if (sum > target) {
            // 左边界向右移动
            sum -= i;
            i++;
        } else {
            // 记录结果
            int[] arr = new int[j-i];
            for (int k = i; k < j; k++) {
                arr[k-i] = k;
            }
            res.add(arr);
            // 右边界向右移动
            sum -= i;
            i++;
        }
    }

    return res.toArray(new int[res.size()][]);
}
 ```

### [0008]左旋转字符串

字符串的左旋转操作是把字符串前面的若干个字符转移到字符串的尾部。请定义一个函数实现字符串左旋转操作的功能。比如，输入字符串"abcdefg"和数字2，该函数将返回左旋转两位得到的结果"cdefgab"。

示例 1：

```
输入: s = "abcdefg", k = 2
输出: "cdefgab"
```


示例 2：

```
输入: s = "lrloseumgh", k = 6
输出: "umghlrlose"
```

限制：

```
1 <= k < s.length <= 10000
```

**方法一：**

 ```java
class Solution {
    public String reverseLeftWords(String s, int n) {
        return s.substring(n) + s.substring(0,n);
    }
}
 ```

 **方法二：翻转reverse**

```java
class Solution {
     public String reverseLeftWords(String s, int n) {
        char[] chars = s.toCharArray();
        reverse(chars, 0, n-1);
        reverse(chars, n, chars.length-1);
        reverse(chars, 0, chars.length-1);
        return new String(chars);
    }
    private void reverse(char[] arr, int i, int j) {
        while(i < j) {
            char x = arr[i];
            arr[i] = arr[j];
            arr[j] = x;
            i++;
            j--;
        }
    }
}
```

- 时间：遍历数组，O(N)
- 空间：新开辟数组，O(N)

### [0009]将数字变成 0 的操作次数

给你一个非负整数 `num` ，请你返回将它变成 0 所需要的步数。 如果当前数字是偶数，你需要把它除以 2 ；否则，减去 1 。

示例 1：

```
输入：num = 14
输出：6
解释：
步骤 1) 14 是偶数，除以 2 得到 7 。
步骤 2） 7 是奇数，减 1 得到 6 。
步骤 3） 6 是偶数，除以 2 得到 3 。
步骤 4） 3 是奇数，减 1 得到 2 。
步骤 5） 2 是偶数，除以 2 得到 1 。
步骤 6） 1 是奇数，减 1 得到 0 。
```


示例 2：

```
输入：num = 8
输出：4
解释：
步骤 1） 8 是偶数，除以 2 得到 4 。
步骤 2） 4 是偶数，除以 2 得到 2 。
步骤 3） 2 是偶数，除以 2 得到 1 。
步骤 4） 1 是奇数，减 1 得到 0 。
```


示例 3：

```
输入：num = 123
输出：12
```

- 奇偶判断。
- 奇数自减1。
  - 对`~1`按位与`&`
  - 直接对`1`按位异或`^`

**方法一：迭代**

```java
public int numberOfSteps(int num) {
    int step = 0;            //记录步数
    while (num > 0) {
        if ((num & 1) == 0) {//num为偶数
            num >>= 1;
        }
        else {               //num为奇数
            num &= ~1;       //等价于 num ^= 1
        }        
        step++;
    }
    return step;
}
```

- 时间复杂度：O(1)，num是个常数。
- 空间复杂度：O(1)，使用常数级别的空间。

**方法二：递归**

```java
public int numberOfSteps1(int num) {
    if(num == 0) 
        return 0;
    if((num & 1) == 0) { // num为偶数
        return numberOfSteps1(num >>> 1) + 1;
    }
    else {               // num为奇数       
        return numberOfSteps1(num - 1) + 1;
    }
}
```

- 时间复杂度：O(1)
- 空间复杂度：O(1)

### [0010]解压缩编码列表

给你一个以行程长度编码压缩的整数列表 nums 。

考虑每对相邻的两个元素` freq, val] = [nums[2*i], nums[2*i+1]] `（其中 i >= 0 ），每一对都表示解压后子列表中有 freq 个值为 val 的元素，你需要从左到右连接所有子列表以生成解压后的列表。

请你返回解压后的列表。 

示例：

```
输入：nums = [1,2,3,4]
输出：[2,4,4,4]
解释：第一对 [1,2] 代表着 2 的出现频次为 1，所以生成数组 [2]。
第二对 [3,4] 代表着 4 的出现频次为 3，所以生成数组 [4,4,4]。
最后将它们串联到一起 [2] + [4,4,4] = [2,4,4,4]。
```


示例 2：

```
输入：nums = [1,1,2,3]
输出：[1,3,3]
```

提示：

```
2 <= nums.length <= 100
nums.length % 2 == 0
1 <= nums[i] <= 100
```



```java
class Solution {
    public int[] decompressRLElist(int[] nums) {
        int length = 0;
        
        for(int i=0;i<nums.length;i+=2){
            length += nums[i];
        }
        int[] result = new int[length];
        // 新数组角标
        int index = 0;
        for(int i = 0; i < nums.length; i+=2){
            // 填充a个b,每填充一次,a-1,index+1
            int a = nums[i];
            while(a > 0){
                result[index] = nums[i+1];
                a--;
                index++;
            }
        }

        return result; 
    }
}
```

- 时间复杂度：O(n)
- 空间复杂度：O(1)

### [0011]整数的各位积和之差

给你一个整数 `n`，请你帮忙计算并返回该整数「各位数字之积」与「各位数字之和」的差。

示例 1：

```
输入：n = 234
输出：15 
解释：
各位数之积 = 2 * 3 * 4 = 24 
各位数之和 = 2 + 3 + 4 = 9 
结果 = 24 - 9 = 15
```


示例 2：

```
输入：n = 4421
输出：21
解释： 
各位数之积 = 4 * 4 * 2 * 1 = 32 
各位数之和 = 4 + 4 + 2 + 1 = 11 
结果 = 32 - 11 = 21
```

解答：

```java
class Solution {
    public int subtractProductAndSum(int n) {
        int add = 0, mul = 1;
        while (n > 0) {
            int digit = n % 10;
            n /= 10;
            add += digit;
            mul *= digit;
        }
        return mul - add;
    }
}
```

- 时间复杂度：O(log⁡N)

- 空间复杂度：O(1)

### [0012]猜数字

小A 和 小B 在玩猜数字。小B 每次从 1, 2, 3 中随机选择一个，小A 每次也从 1, 2, 3 中选择一个猜。他们一共进行三次这个游戏，请返回 小A 猜对了几次？

 

输入的guess数组为 小A 每次的猜测，answer数组为 小B 每次的选择。guess和answer的长度都等于3。

示例 1：

```
输入：guess = [1,2,3], answer = [1,2,3]
输出：3
解释：小A 每次都猜对了。
```

示例 2：

```
输入：guess = [2,2,3], answer = [3,2,1]
输出：1
解释：小A 只猜对了第二次。
```

限制：

```
guess的长度 = 3
answer的长度 = 3
guess的元素取值为 {1, 2, 3} 之一。
answer的元素取值为 {1, 2, 3} 之一。
```

解答：

```java
class Solution {
    public int game(int[] guess, int[] answer) {
        int n=0;//猜中的个数
        for(int i=0;i<3;i++)
            if(guess[i]==answer[i]) 
                n++;
        return n;
    }
}
```

### [0013]统计位数为偶数的数字

给你一个整数数组 nums，请你返回其中位数为 偶数 的数字的个数。 

示例 1：

```
输入：nums = [12,345,2,6,7896]
输出：2
解释：
12 是 2 位数字（位数为偶数） 
345 是 3 位数字（位数为奇数）  
2 是 1 位数字（位数为奇数） 
6 是 1 位数字 位数为奇数） 
7896 是 4 位数字（位数为偶数）  
因此只有 12 和 7896 是位数为偶数的数字
```

示例 2：

```
输入：nums = [555,901,482,1771]
输出：1 
解释： 
只有 1771 是位数为偶数的数字。
```

提示：

```
1 <= nums.length <= 500
1 <= nums[i] <= 10^5
```

方法一：枚举 + 字符串

```java
class Solution {
    public int findNumbers(int[] nums) {
        int result = 0;
        for(int i = 0; i<nums.length;i++){
           result += String.valueOf(nums[i]).length() % 2 == 0 ? 1 : 0;
        }
        return result;
    }
}
```

时间复杂度：O(N)。这里假设将整数转换为字符串的时间复杂度为 O(1)。

空间复杂度：O(1)

 方法二：范围已知

```java
class Solution {
    public int findNumbers(int[] nums) {
        int result = 0;
        for(int i = 0; i<nums.length;i++){       
           if((nums[i]>=10&&nums[i]<100)||(nums[i]>=1000&&nums[i]<10000))
               result++;
        }
        return result;
    }
}
```

时间复杂度：O(N)

空间复杂度：O(1)

 方法三：枚举 + 数学

```java
class Solution {
    public int findNumbers(int[] nums) {
        int result = 0;
         for(int num:nums){
             //logx(y) =loge(y) / loge(x)
             if ((int)(Math.log(num)/Math.log(10) + 1) % 2 == 0) 
                result++;
        }
        return result;
    }
}
```

时间复杂度：O(N)

空间复杂度：O(1)

### [0014]宝石与石头

 给定字符串J 代表石头中宝石的类型，和字符串 S代表你拥有的石头。 S 中每个字符代表了一种你拥有的石头的类型，你想知道你拥有的石头中有多少是宝石。

J 中的字母不重复，J 和 S中的所有字符都是字母。字母区分大小写，因此"a"和"A"是不同类型的石头。

示例 1:

```
输入: J = "aA", S = "aAAbbbb"
输出: 3
示例 2:

输入: J = "z", S = "ZZ"
输出: 0
注意:
```

S 和 J 最多含有50个字母。
 J 中的字符不重复。

方法一： 暴力法 

```java
class Solution {
    public int numJewelsInStones(String J, String S) {
        int ans = 0;
        for (char s: S.toCharArray()) // For each stone...
            for (char j: J.toCharArray()) // For each jewel...
                if (j == s) {  // If the stone is a jewel...
                    ans++;
                    break; // Stop searching whether this stone 's' is a jewel
                }
        return ans;
    }
}
```

时间复杂度：O(J.length * S.length))。

空间复杂度：在 Java 实现中，空间复杂度为 O(J.length∗S.length))。

方法二： 哈希集合

```java
class Solution {
    public int numJewelsInStones(String J, String S) {
         Set<Character> Jset = new HashSet();
        for (char j: J.toCharArray())
            Jset.add(j);

        int ans = 0;
        for (char s: S.toCharArray())
            if (Jset.contains(s))
                ans++;
        return ans;
    }
}
```

时间复杂度：O(J.length + S.length))。

空间复杂度：O(J.length)

方法三：位运算

解决类似 Set 或 boolean[] 的、表示有或无二选一情况时都可考虑

注意共多少种情况，此处 int 的32位不够

```java
class Solution {
    public int numJewelsInStones(String J, String S) {
        long jewels = 0b0L;
        for (char c : J.toCharArray()) jewels |= 1L << (c - 'A');

        int count = 0;
        for (char c : S.toCharArray()) count += (jewels >> (c - 'A')) & 1;
        return count;
    }
}
```

方法四：byte

```java
class Solution {
    public int numJewelsInStones(String J, String S) {
        byte[] arr = new byte[58];
        int count = 0;
        for (char ch : J.toCharArray()) {
            arr[ch - 65] = 1;
        }
        for (char ch : S.toCharArray()) {
            if(arr[ch -65] == 1) {
                count++;
            };
        }
        return count;
    }
}
```



### [0015]IP 地址无效化

给你一个有效的 IPv4 地址 address，返回这个 IP 地址的无效化版本。

所谓无效化 IP 地址，其实就是用 "[.]" 代替了每个 "."。

 

示例 1：

```java
输入：address = "1.1.1.1"
输出："1[.]1[.]1[.]1"
```

示例 2：

```java
输入：address = "255.100.50.0"
输出："255[.]100[.]50[.]0"
```


提示：

给出的 address 是一个有效的 IPv4 地址

方法一：

```java
class Solution {
    public String defangIPaddr(String address) {
        return address.replace(".","[.]");
    }
}	
```

方法二：

```java
class Solution {
    public String defangIPaddr(String address) {
          StringBuilder s = new StringBuilder(address);
        for (int i = 0; i < s.length(); i++) {
            if (s.charAt(i) == '.') {
                s.insert(i + 1, ']');// 先插入后面，此时 i 下标仍是'.'
                s.insert(i, '[');// 插入 '.' 前面，此时 i 下标是'[' ,i+2 下标为']'
                i += 3;// 故 i 直接加 3，为下一个字符，注意此时已经是原来 i+1 下标的字符了；
                //此次循环结束进入下次循环还会进行加 1，不过又因为 ip 地址格式的原因，不会有连续的两个 '.' 连着；
                //所以这个位置绝不可能是 '.'，所以再加 1，也没问题。
            }
        }
        return s.toString();
    }
}
```

方法三：

```java
class Solution {
    public String defangIPaddr(String address) {
        StringBuilder s = new StringBuilder();
        for (int i = 0; i < address.length(); i++) {
            if (address.charAt(i) == '.') {
                s.append("[.]");
            } else {
                s.append(address.charAt(i));
            }
        }
        return s.toString();
    }
}
```

### [0016]TinyURL 的加密与解密

TinyURL是一种URL简化服务， 比如：当你输入一个URL https://leetcode.com/problems/design-tinyurl 时，它将返回一个简化的URL http://tinyurl.com/4e9iAk.

要求：设计一个 TinyURL 的加密 encode 和解密 decode 的方法。你的加密和解密算法如何设计和运作是没有限制的，你只需要保证一个URL可以被加密成一个TinyURL，并且这个TinyURL可以用解密方法恢复成原本的URL。



方法 1：使用简单的计数 

为了加密 URL，我们使用计数器 (i) ，每遇到一个新的 URL 都加一。我们将 URL 与它的次数 i 放在哈希表 HashMap 中，这样我们在稍后的解密中可以轻易地获得原本的 URL。

```java
public class Codec {

     Map<Integer, String> map = new HashMap<>();
    int i = 0;

    public String encode(String longUrl) {
        map.put(i, longUrl);
        return "http://tinyurl.com/" + i++;
    }

    public String decode(String shortUrl) {
        return map.get(Integer.parseInt(shortUrl.replace("http://tinyurl.com/", "")));
    }

}
```

表现分析

可以加密解密的 URL 数目受限于 int 所能表示的范围。

如果超过 int 个 URL 需要被加密，那么超过范围的整数会覆盖之前存储的 URL，导致算法失效。

URL 的长度不一定比输入的 longURL 短。它只与加密的 URL 被加密的顺序有关。

这个方法的问题是预测下一个会产生的加密 URL 非常容易，因为产生几个 URL 后很容易推测出生成的模式。



方法 2：使用出现次序加密

算法

这种方法中，我们将当前 URL 第几个出现作为关键字进行加密，将这个出现次序看做 62 进制，并将每一位映射到一个长度为 62 位的表中对应的字母作为哈希值。此方法中，我们使用一系列整数和字母表来加密，而不是仅仅使用数字进行加密。

```java
public class Codec {

    String chars = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
    HashMap<String, String> map = new HashMap<>();
    int count = 1;

    public String getString() {
        int c = count;
        StringBuilder sb = new StringBuilder();
        while (c > 0) {
            c--;
            sb.append(chars.charAt(c % 62));
            c /= 62;
        }
        return sb.toString();
    }

    public String encode(String longUrl) {
        String key = getString();
        map.put(key, longUrl);
        count++;
        return "http://tinyurl.com/" + key;
       
    }

    public String decode(String shortUrl) {
        return map.get(shortUrl.replace("http://tinyurl.com/", ""));
    }
}
```

表现分析

可加密的 URL 数目还是依赖于int 的范围。因为相同的 count 在出现次序溢出整数范围后仍然会出现。

加密后 URL 的长度不一定更短，但某种程度上与longURL 的出现次序相对独立。比方说产生的 URL 长度按顺序会是 1（62次），2（62次）。

这个算法的表现比较好，因为相同的加密结果只有在溢出整数后才会发生，这个范围非常大。

如果出现重复，下一次产生的加密结果还是能通过某种计算被预测出来。



方法 3：使用hashcode 

这种方法中，我们使用一种内建函数hashCode() 来为每一个 URL 产生加密结果。同样的，映射结果保存在 HashMap 中以供解码。

```java
public class Codec {

    Map<Integer, String> map = new HashMap<>();

    public String encode(String longUrl) {
        map.put(longUrl.hashCode(), longUrl);
        return "http://tinyurl.com/" + longUrl.hashCode();
    }

    public String decode(String shortUrl) {
        return map.get(Integer.parseInt(shortUrl.replace("http://tinyurl.com/", "")));
    }
}
```

表现分析

可加密 URL 的数目由 int 决定，因为 hashCode 使用整数运算。

加密后 URL 的平均长度与 longURL 的长度没有直接关联。

hashCode()对于不同的字符串不一定产生独一无二的加密后 URL。像这样对于不同输入产生相同输出的过程叫做冲突。因此，如果加密字符串的数目增加，冲突的概率也会增加，最终导致算法失效。

可能几个字符串加密后冲突就会发生，会远比 int 要小。这与生日悖论类似，也就是如果有23个人，存在 2 个人同一天生日的概率达到 50%，如果有 70 个人，这一概率会高达 99.9%。

这种方法中，很难根据前面产生的 URL 结果预测后面加密 URL 的答案。

方法 4：使用随机数 [Accepted]

算法

这个方法中，我们使用随机整数来加密。为了防止产生的结果与之前某个 longURL 产生的结果相同，我们生成一个新的随机数作为加密结果。这个数据存在哈希表 HashMap 中，以便解码。

```java
public class Codec {
    Map<Integer, String> map = new HashMap<>();
    Random r = new Random();
    int key = r.nextInt(Integer.MAX_VALUE);

    public String encode(String longUrl) {
        while (map.containsKey(key)) {
            key = r.nextInt(Integer.MAX_VALUE);
        }
        map.put(key, longUrl);
        return "http://tinyurl.com/" + key;
    }

    public String decode(String shortUrl) {
        return map.get(Integer.parseInt(shortUrl.replace("http://tinyurl.com/", "")));
    }
}
```

表现分析

能被加密的 URL 数目受限于 int。

加密 URL 的平均长度与 longURL 的长度无关，因为使用了随机整数。

URL 的长度不一定比输入的 longURL 短。只与 URL 加密的相对顺序有关。

由于加密过程中使用了随机数，就像前面的算法所述，当输入字符串的数目增加时，冲突的次数也会增加，导致算法失效。

由于使用了随机数，想根据产生的 URL 推测出加密算法是不可能的。

方法 5：随机固定长度加密 

算法

在这种方法中，我们像方法 2 一样再次使用数字和字母表集合来为 URL 生成加密结果。这种方法中，加密后的长度固定是 6 位。如果产生出来的加密结果与之前产生的结果一样，就换一个新的加密结果。

```java
public class Codec {
    String alphabet = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
    HashMap<String, String> map = new HashMap<>();
    Random rand = new Random();
    String key = getRand();

    public String getRand() {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 6; i++) {
            sb.append(alphabet.charAt(rand.nextInt(62)));
        }
        return sb.toString();
    }

    public String encode(String longUrl) {
        while (map.containsKey(key)) {
            key = getRand();
        }
        map.put(key, longUrl);
        return "http://tinyurl.com/" + key;
    }

    public String decode(String shortUrl) {
        return map.get(shortUrl.replace("http://tinyurl.com/", ""));
    }
}
```

表现分析

可加密的 URL 数目非常大

加密 URL 的长度固定是 6，这相比于能加密的字符串数目是极大的缩减优化。

这个方法的表现非常好，因为几乎不可能产生相同加密结果。

我们也可以通过增加加密字符串的长度来增加加密结果的数目。因此，在加密字符串的长度和可加密的字符串数目之间我们需要做一个权衡。

