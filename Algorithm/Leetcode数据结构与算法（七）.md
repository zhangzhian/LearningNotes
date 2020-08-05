# Leetcode数据结构与算法

### [0097] 一维数组的动态和

给你一个数组 nums 。数组「动态和」的计算公式为：runningSum[i] = sum(nums[0]…nums[i]) 。

请返回 nums 的动态和。

 示例 1：

```
输入：nums = [1,2,3,4]
输出：[1,3,6,10]
解释：动态和计算过程为 [1, 1+2, 1+2+3, 1+2+3+4] 。
```


示例 2：

```
输入：nums = [1,1,1,1,1]
输出：[1,2,3,4,5]
解释：动态和计算过程为 [1, 1+1, 1+1+1, 1+1+1+1, 1+1+1+1+1] 。
```


示例 3：

```
输入：nums = [3,1,2,10,1]
输出：[3,4,6,16,17]
```

提示：

```
1 <= nums.length <= 1000
-10^6 <= nums[i] <= 10^6
```

方法一：

```java
class Solution {
    public int[] runningSum(int[] nums) {
        int[] ret = new int[nums.length];
        for(int i= 0;i<nums.length;i++){
           ret[i] = (i == 0 ? nums[0] : nums[i]+ret[i-1]);
        }
        return ret;
    }
}
```

### [0098] 好数对的数目

给你一个整数数组 nums 。

如果一组数字 (i,j) 满足 nums[i] == nums[j] 且 i < j ，就可以认为这是一组 好数对 。

返回好数对的数目。

示例 1：

```
输入：nums = [1,2,3,1,1,3]
输出：4
解释：有 4 组好数对，分别是 (0,3), (0,4), (3,4), (2,5) ，下标从 0 开始
```


示例 2：

```
输入：nums = [1,1,1,1]
输出：6
解释：数组中的每组数字都是好数对
```


示例 3：

```
输入：nums = [1,2,3]
输出：0
```

提示：

```
1 <= nums.length <= 100
1 <= nums[i] <= 100
```

方法一：

```java
class Solution {
    public int numIdenticalPairs(int[] nums) {
        int ret = 0;
        for(int i=0;i<nums.length;i++)
            for(int j=i+1;j<nums.length;j++)
                if(nums[i] == nums[j]) ret++;
        return ret;
    }
}
```

- 时间复杂度：O(n^2))。
- 空间复杂度：O(1)。

方法二：
假如输入[1,1,1,1]。我们从前往后遍历的时候，好数对的数量为：

3: [0,1],[0,2],[0,3]

2: [1,2],[1,3]

1: [2,3]

这里我们举的例子是[1,1,1,1]，如果我们的例子是[2,1,1,1,1,4]结果也是一样的，所以当我们从前往后遍历的时候，我们把数值存放在temp数组中，实际上计算的结果顺序是从后往前的，即1 + 2 + 3 = 6。这个结果其实跟我们正常理解的3 + 2 + 1 = 6的结果是一致的。

```java
class Solution {
    public int numIdenticalPairs(int[] nums) {
        int[] temp = new int[101];
        int ans = 0;
        for(int num : nums){
            //temp[num]存放的就是满足nums[i] == nums[j]的数目
            temp[num]++;
            //temp[num]-1理解为：满足nums[i] == nums[j]的好数对的数目。只不过这里的求值过程为倒序的
            ans+=temp[num]-1;
        }
        return ans;
    }
}
```

```java
class Solution {
    public int numIdenticalPairs(int[] nums) {
        int[] temps = new int[101];
        int ans = 0;
        for(int num : nums){
            temps[num]++;
        }
        for(int temp : temps){
            if(temp != 0) ans += temp * (temp - 1) /2;
        }
        return an;
    }
}
```

- 时间复杂度：O(n)。
- 空间复杂度：O(n)。

### [0099] 数组异或操作

给你两个整数，n 和 start 。

数组 nums 定义为：nums[i] = start + 2*i（下标从 0 开始）且 n == nums.length 。

请返回 nums 中所有元素按位异或（XOR）后得到的结果。

 

示例 1：

```
输入：n = 5, start = 0
输出：8
解释：数组 nums 为 [0, 2, 4, 6, 8]，其中 (0 ^ 2 ^ 4 ^ 6 ^ 8) = 8 。
     "^" 为按位异或 XOR 运算符。
```

示例 2：

```
输入：n = 4, start = 3
输出：8
解释：数组 nums 为 [3, 5, 7, 9]，其中 (3 ^ 5 ^ 7 ^ 9) = 8.
```


示例 3：

```
输入：n = 1, start = 7
输出：7
```


示例 4：

```
输入：n = 10, start = 5
输出：2
```


提示：

```
1 <= n <= 1000
0 <= start <= 1000
n == nums.length
```

方法一：

```java
class Solution {
    public int xorOperation(int n, int start) {
        int ans = 0;
        
        for(int i = 0; i < n; i++) {
            int elem = start + 2 * i;
            ans = ans ^ elem;
        }
        
        return ans;
    }
}
```

方法二：位运算（利用 XOR 的特性）

https://leetcode-cn.com/problems/xor-operation-in-an-array/solution/o1-wei-yun-suan-by-bruceyuj/

```java
class Solution {
    public int xorOperation(int n, int start) {
        int ans = 2 * xor(n, (int)Math.floor(start/2));
        
        if((n & start & 1) != 0) ans++; // 处理最后一位
        
        return ans;
    }

    public int xor(int n, int start) { // 将公式转换成情况 1
        if((start & 1)!=0) return (start-1) ^ helper(n+1, start-1);
        else return helper(n, start);
    }
    
    public int helper(int n, int start) { // 情况 1
        if(n % 2 == 0) return (int)Math.floor(n/2) & 1;
        else return (int)Math.floor(n/2) & 1 ^ (start+n-1);
    }
}
```

### [0100] 判断能否形成等差数列

给你一个数字数组 arr 。

如果一个数列中，任意相邻两项的差总等于同一个常数，那么这个数列就称为 等差数列 。

如果可以重新排列数组形成等差数列，请返回 true ；否则，返回 false 。

 示例 1：

```
输入：arr = [3,5,1]
输出：true
解释：对数组重新排序得到 [1,3,5] 或者 [5,3,1] ，任意相邻两项的差分别为 2 或 -2 ，可以形成等差数列。
```

示例 2：

```
输入：arr = [1,2,4]
输出：false
解释：无法通过重新排序得到等差数列。
```

方法一：

```java
class Solution {
    public boolean canMakeArithmeticProgression(int[] arr) {
        Arrays.sort(arr);
        int d = arr[1] - arr[0];
        for (int i = 2; i < arr.length; i++) {
            if (arr[i] - arr[i - 1] != d) return false;
        }
        return true;
    }
}
```

### [0101] 换酒问题

小区便利店正在促销，用 numExchange 个空酒瓶可以兑换一瓶新酒。你购入了 numBottles 瓶酒。

如果喝掉了酒瓶中的酒，那么酒瓶就会变成空的。

请你计算 最多 能喝到多少瓶酒。

 示例 1：

**![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/07/19/sample_1_1875.png)**

```
输入：numBottles = 9, numExchange = 3
输出：13
解释：你可以用 3 个空酒瓶兑换 1 瓶酒。
所以最多能喝到 9 + 3 + 1 = 13 瓶酒。
```

示例 2：

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/07/19/sample_2_1875.png)

```
输入：numBottles = 15, numExchange = 4
输出：19
解释：你可以用 4 个空酒瓶兑换 1 瓶酒。
所以最多能喝到 15 + 3 + 1 = 19 瓶酒。
```


示例 3：

```
输入：numBottles = 5, numExchange = 5
输出：6
```

示例 4：

```
输入：numBottles = 2, numExchange = 3
输出：2
```


提示：

```
1 <= numBottles <= 100
2 <= numExchange <= 100
```

方法一：

```java
class Solution {
    public int numWaterBottles(int numBottles, int numExchange) {
        int ret = numBottles;
        int temp1 = numBottles/numExchange; 
        int temp2 = numBottles%numExchange; 
        int tmp = temp1 + temp2;
        while(tmp >= numExchange){
            ret += temp1;
                
            temp1 = tmp / numExchange; 
            temp2 = tmp % numExchange;
            tmp = temp1 + temp2; 
        }
        ret += temp1;    
        return ret;
    }
}
```

方法二：

```
class Solution {
    public int numWaterBottles(int numBottles, int numExchange) {
        int bottle = numBottles, ans = numBottles;
        while (bottle >= numExchange) {
            bottle -= numExchange;
            ++ans;
            ++bottle;
        }
        return ans;
    }
}
```

方法三：(数学解法)
https://leetcode-cn.com/problems/water-bottles/solution/huan-jiu-wen-ti-by-leetcode-solution/

```
class Solution {
    public int numWaterBottles(int numBottles, int numExchange) {
        return numBottles >= numExchange ? (numBottles - numExchange) / (numExchange - 1) + 1 + numBottles : numBottles;
    }
}
```

### [0102] 除数博弈

爱丽丝和鲍勃一起玩游戏，他们轮流行动。爱丽丝先手开局。

最初，黑板上有一个数字 N 。在每个玩家的回合，玩家需要执行以下操作：

选出任一 x，满足 0 < x < N 且 N % x == 0 。

用 N - x 替换黑板上的数字 N 。

如果玩家无法执行这些操作，就会输掉游戏。

只有在爱丽丝在游戏中取得胜利时才返回 True，否则返回 False。假设两个玩家都以最佳状态参与游戏。

示例 1：

```
输入：2
输出：true
解释：爱丽丝选择 1，鲍勃无法进行操作。
```

示例 2：

```
输入：3
输出：false
解释：爱丽丝选择 1，鲍勃也选择 1，然后爱丽丝无法进行操作。
```


提示：

```
1 <= N <= 1000
```

方法一：找规律

N = 1 的时候，区间 (0, 1)(0,1) 中没有整数是 n 的因数，所以此时 Alice 败。
N = 2 的时候，Alice 只能拿 1，N 变成 1，Bob 无法继续操作，故 Alice 胜。
N = 3 的时候，Alice 只能拿 1，N 变成 2，根据 N = 2 的结论，我们知道此时 Bob 会获胜，Alice 败。
N = 4 的时候，Alice 能拿 1 或 2，如果 Alice 拿 1，根据 N = 3 的结论，Bob 会失败，Alice 会获胜。
N = 5 的时候，Alice 只能拿 1，根据 N = 4 的结论，Alice 会失败。
......
N 为奇数的时候 Alice（先手）必败，N 为偶数的时候 Alice 必胜。下面想办法证明它。

证明:

N=1 和 N = 2 时结论成立。

N > 2时，假设 N≤k 时该结论成立，则 N = k + 1 时：

如果 k 为偶数，则 k + 1为奇数，x 是 k+1 的因数，只可能是奇数，而奇数减去奇数等于偶数，且 kk+1−x≤k，故轮到 Bob 的时候都是偶数。而根据我们的猜想假设 N≤k 的时候偶数的时候先手必胜，故此时无论 Alice 拿走什么，Bob 都会处于必胜态，所以 Alice 处于必败态。
如果 k 为奇数，则k+1 为偶数，x 可以是奇数也可以是偶数，若 Alice 减去一个奇数，那么 k + 1 - x 是一个小于等于 k 的奇数，此时 Bob 占有它，处于必败态，则 Alice 处于必胜态。

综上所述，这个猜想是正确的。

```java
class Solution {
    public boolean divisorGame(int N) {
        return N % 2 == 0;
    }
}
```

方法二：递推

Alice 处在 N = k 的状态时，他（她）做一步操作，必然使得 Bob 处于 N = m (m < k) 的状态。因此我们只要看是否存在一个 m 是必败的状态，那么 Alice 直接执行对应的操作让当前的数字变成 m，Alice 就必胜了，如果没有任何一个是必败的状态的话，说明 Alice 无论怎么进行操作，最后都会让 Bob 处于必胜的状态，此时 Alice 是必败的。

结合以上我们定义 f[i] 表示当前数字 ii 的时候先手是处于必胜态还是必败态，true 表示先手必胜，false 表示先手必败，从前往后递推，根据我们上文的分析，枚举 i 在 (0, i) 中 i 的因数 j，看是否存在 f[i−j] 为必败态即可。

```java
class Solution {
    public boolean divisorGame(int N) {
        boolean[] f = new boolean[N + 5];

        f[1] = false;
        f[2] = true;
        for (int i = 3; i <= N; ++i) {
            for (int j = 1; j < i; ++j) {
                if ((i % j) == 0 && !f[i - j]) {
                    f[i] = true;
                    break;
                }
            }
        }

        return f[N];
    }
}
```

### [0103] 移除重复节点

编写代码，移除未排序链表中的重复节点。保留最开始出现的节点。

示例1:

```
 输入：[1, 2, 3, 3, 2, 1]
 输出：[1, 2, 3]
```

示例2:

```
 输入：[1, 1, 1, 1, 2]
 输出：[1, 2]
```


提示：

```
链表长度在[0, 20000]范围内。
链表元素在[0, 20000]范围内。
```

进阶：

```
如果不得使用临时缓冲区，该怎么解决？
```

方法一：哈希表

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
    public ListNode removeDuplicateNodes(ListNode head) {
        if (head == null) {
            return head;
        }
        Set<Integer> occurred = new HashSet<Integer>();
        occurred.add(head.val);
        ListNode pos = head;
        // 枚举前驱节点
        while (pos.next != null) {
            // 当前待删除节点
            ListNode cur = pos.next;
            if (occurred.add(cur.val)) {
                pos = pos.next;
            } else {
                pos.next = pos.next.next;
            }
        }
        pos.next = null;
        return head;
    }
}
```

方法二：两重循环(不得使用临时缓冲区)

```java
class Solution {
    public ListNode removeDuplicateNodes(ListNode head) {
        ListNode ob = head;
        while (ob != null) {
            ListNode oc = ob;
            while (oc.next != null) {
                if (oc.next.val == ob.val) {
                    oc.next = oc.next.next;
                } else {
                    oc = oc.next;
                }
            }
            ob = ob.next;
        }
        return head;
    }
}
```

### [0104] 配对交换

配对交换。编写程序，交换某个整数的奇数位和偶数位，尽量使用较少的指令（也就是说，位0与位1交换，位2与位3交换，以此类推）。

示例1:

```
 输入：num = 2（或者0b10）
 输出 1 (或者 0b01)
```

示例2:

```
 输入：num = 3
 输出：3
```


提示:

```
num的范围在[0, 2^30 - 1]之间，不会发生整数溢出。
```

方法一：

```java
class Solution {
    public int exchangeBits(int num) {
        //取奇数位
        int odd = num & 0x55555555;	//0x55555555 = 0b0101_0101_0101_0101_0101_0101_0101_0101
        //取偶数位
        int even = num & 0xaaaaaaaa;//0xaaaaaaaa = 0b1010_1010_1010_1010_1010_1010_1010_1010
        odd = odd << 1;		// 左移一位			
        even = even >>> 1; 	// 右移一位 >>> 无符号右移，高位补0
        return odd | even;
    }
}
```

### [0105] 位1的个数

编写一个函数，输入是一个无符号整数，返回其二进制表达式中数字位数为 ‘1’ 的个数（也被称为汉明重量）。 

示例 1：

```
输入：00000000000000000000000000001011
输出：3
解释：输入的二进制串 00000000000000000000000000001011 中，共有三位为 '1'。
```

示例 2：

```
输入：00000000000000000000000010000000
输出：1
解释：输入的二进制串 00000000000000000000000010000000 中，共有一位为 '1'。
```

示例 3：

```
输入：11111111111111111111111111111101
输出：31
解释：输入的二进制串 11111111111111111111111111111101 中，共有 31 位为 '1'。
```


提示：

```
请注意，在某些语言（如 Java）中，没有无符号整数类型。在这种情况下，输入和输出都将被指定为有符号整数类型，并且不应影响您的实现，因为无论整数是有符号的还是无符号的，其内部的二进制表示形式都是相同的。
在 Java 中，编译器使用二进制补码记法来表示有符号整数。因此，在上面的 示例 3 中，输入表示有符号整数 -3。
```

方法一：

```java
public class Solution {
    public int hammingWeight(int n) {
        int bits = 0;
        int mask = 1;
        for (int i = 0; i < 32; i++) {
            if ((n & mask) != 0) {
                bits++;
            }
            mask <<= 1;
        }
        return bits;
    }
}
```

方法二：

不断把数字最后一个 11 反转，并把答案加一。当数字变成 00 的时候偶，我们就知道它没有 11 的位了，此时返回答案。

对于任意数字 n ，将 n和 n - 1 做与运算，会把最后一个 1 的位变成 0 。

![image.png](https://pic.leetcode-cn.com/abfd6109e7482d70d20cb8fc1d632f90eacf1b5e89dfecb2e523da1bcb562f66-image.png)

```java
public class Solution {
    public int hammingWeight(int n) {
        int sum = 0;
        while (n != 0) {
            sum++;
            n &= (n - 1);
        }
        return sum;
    }
}
```

### [0106] 二叉树的最近公共祖先

给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

百度百科中最近公共祖先的定义为：“对于有根树 T 的两个结点 p、q，最近公共祖先表示为一个结点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（一个节点也可以是它自己的祖先）。”

例如，给定如下二叉树:  root = [3,5,1,6,2,0,8,null,null,7,4]

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/15/binarytree.png)

示例 1:

```
输入: root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 1
输出: 3
解释: 节点 5 和节点 1 的最近公共祖先是节点 3。
```

示例 2:

```
输入: root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 4
输出: 5
解释: 节点 5 和节点 4 的最近公共祖先是节点 5。因为根据定义最近公共祖先节点可以为节点本身。
```


说明:

```
所有节点的值都是唯一的。
p、q 为不同节点且均存在于给定的二叉树中。
```

方法一：后序遍历 DFS 

[分析详情](https://leetcode-cn.com/problems/er-cha-shu-de-zui-jin-gong-gong-zu-xian-lcof/solution/mian-shi-ti-68-ii-er-cha-shu-de-zui-jin-gong-gon-7/)

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if(root == null || root == p || root == q) return root;
        TreeNode left = lowestCommonAncestor(root.left, p, q);
        TreeNode right = lowestCommonAncestor(root.right, p, q);
        if(left == null) return right;
        if(right == null) return left;
        return root;
    }
}
```

### [0107] 最长特殊序列 Ⅰ

给你两个字符串，请你从这两个字符串中找出最长的特殊序列。

「最长特殊序列」定义如下：该序列为某字符串独有的最长子序列（即不能是其他字符串的子序列）。

子序列 可以通过删去字符串中的某些字符实现，但不能改变剩余字符的相对顺序。空序列为所有字符串的子序列，任何字符串为其自身的子序列。

输入为两个字符串，输出最长特殊序列的长度。如果不存在，则返回 -1。

 示例 1：

```
输入: "aba", "cdc"
输出: 3
解释: 最长特殊序列可为 "aba" (或 "cdc")，两者均为自身的子序列且不是对方的子序列。
```

示例 2：

```
输入：a = "aaa", b = "bbb"
输出：3
```

示例 3：

```
输入：a = "aaa", b = "aaa"
输出：-1
```


提示：

```
两个字符串长度均处于区间 [1 - 100] 。
字符串中的字符仅含有 'a'~'z' 。
```

方法一：简单解法 

字符串 a 和 b 共有 3 种情况：

- a=b。如果两个字符串相同，则没有特殊子序列，返回 -1。

- length(a) = length(b)且 a!=b。例如：abc 和 abd。这种情况下，一个字符串一定不会是另外一个字符串的子序列，因此可以将任意一个字符串看作是特殊子序列，返回 length(a) 或 length(b)。

- length(a) != length(b)。例如：abcd 和 abc。这种情况下，长的字符串一定不会是短字符串的子序列，因此可以将长字符串看作是特殊子序列，返回 max(length(a),length(b))。

```java
public class Solution {
    public int findLUSlength(String a, String b) {
        if (a.equals(b))
            return -1;
        return Math.max(a.length(), b.length());
    }
}。
```

### [0108] 玩筹码

数轴上放置了一些筹码，每个筹码的位置存在数组 chips 当中。

你可以对 任何筹码 执行下面两种操作之一（不限操作次数，0 次也可以）：

将第 i 个筹码向左或者右移动 2 个单位，代价为 0。
将第 i 个筹码向左或者右移动 1 个单位，代价为 1。
最开始的时候，同一位置上也可能放着两个或者更多的筹码。

返回将所有筹码移动到同一位置（任意位置）上所需要的最小代价。

 示例 1：

```
输入：chips = [1,2,3]
输出：1
解释：第二个筹码移动到位置三的代价是 1，第一个筹码移动到位置三的代价是 0，总代价为 1。
```

示例 2：

```
输入：chips = [2,2,2,3,3]
输出：2
解释：第四和第五个筹码移动到位置二的代价都是 1，所以最小总代价为 2。
```

方法一：

因为移动2个位置不需要代价，那么奇数位置移到奇数位置不用代价，偶数位置移到偶数位置不用代价，那就分别统计奇数位置和偶数位置的个数，相当于把所有奇数放一起，所有偶数的放一起，然后比较奇数的少还是偶数的少，将少的个数移到多的个数位置上去就可以了。

````java
class Solution {
    public int minCostToMoveChips(int[] chips) {
        int odd = 0, even = 0;
        for (int i = 0; i < chips.length; i++) {
            if (chips[i] % 2 == 0) {
                even++;
            } else {
                odd++;
            }
        }
        return Math.min(even, odd);
    }
}
````

