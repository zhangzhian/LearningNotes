# 剑指offer：中等部分

### [001] 求1+2+…+n

求 1+2+...+n ，要求不能使用乘除法、for、while、if、else、switch、case等关键字及条件判断语句（A?B:C）。

示例 1：

```
输入: n = 3
输出: 6
```

示例 2：

```
输入: n = 9
输出: 45
```


限制：

- 1 <= n <= 10000

方法一：平均计算

**问题：** 此计算必须使用 **乘除法** ，因此本方法不可取。

```java
public int sumNums(int n) {
    return (1 + n) * n / 2;
}
```

**思考：**`/ 2`可以用`>>`替代，`*n`可以用什么替代？

考虑 A 和 B 两数相乘的时候我们如何利用加法和位运算来模拟，其实就是将 B 二进制展开，如果 B 的二进制表示下第 i 位为 1，那么这一位对最后结果的贡献就是 `A∗(1<<i)` ，即 `A<<i`。我们遍历 B 二进制展开下的每一位，累加起来就是最后的答案，这个方法也被称作「俄罗斯农民乘法」

```c++
int quickMulti(int A, int B) {
    int ans = 0;
    for ( ; B; B >>= 1) {
        if (B & 1) {
            ans += A;
        }
        A <<= 1;
    }
    return ans;
}
```

上面的 C++ 实现里我们还是需要循环语句，替换循环语句需要自己手动展开，因为题目数据范围 n 为 `[1,10000]`，所以 n 二进制展开最多不会超过 14 位，我们手动展开 14 层代替循环

```java
class Solution {
    public int sumNums(int n) {
        int ans = 0, A = n, B = n + 1;
        boolean flag;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        return ans >> 1;
    }
}
```

方法二： 迭代

**问题**： 循环必须使用 **while 或 fo**r，因此本方法不可取，直接排除。

```java
public int sumNums(int n) {
    int res = 0;
    for(int i = 1; i <= n; i++)
        res += i;
    return res;
}
```

方法三： 递归（推荐）

**问题：** 终止条件需要使用 if，因此本方法不可取。

```java
public int sumNums(int n) {
    if(n == 1) return 1;
    n += sumNums(n - 1);
    return n;
}
```

**思考：** 除了 if 和 switch 等判断语句外，是否有其他方法可用来终止递归？

**逻辑运算符的短路效应：**

常见的逻辑运算符有三种，即 “与 \&\& ”，“或 ||”，“非 ! ” ；而其有重要的短路效应，如下所示：

```java
if(A && B)  // 若 A 为 false ，则 B 的判断不会执行（即短路），直接判定 A && B 为 false

if(A || B) // 若 A 为 true ，则 B 的判断不会执行（即短路），直接判定 A || B 为 true
```

本题需要实现 “当 n = 1 时终止递归” 的需求，可通过短路效应实现。

```java
n > 1 && sumNums(n - 1) // 当 n = 1 时 n > 1 不成立 ，此时 “短路” ，终止后续递归
```

```java
class Solution {
    int res = 0;
    public int sumNums(int n) {
        boolean x = n > 1 && sumNums(n - 1) > 0;
        res += n;
        return res;
    }
}
```

复杂度分析：

- 时间复杂度 O(n) ： 计算 n + (n-1) + ... + 2 + 1 需要开启 n 个递归函数。
- 空间复杂度 O(n) ： 递归深度达到 n ，系统使用 O(n) 大小的额外空间。

### [002] 数组中数字出现的次数 II

在一个数组 nums 中除一个数字只出现一次之外，其他数字都出现了三次。请找出那个只出现一次的数字。

示例 1：

```
输入：nums = [3,4,3,3]
输出：4
```

示例 2：

```
输入：nums = [9,1,7,9,7,9,7]
输出：1
```


限制：

- 1 <= nums.length <= 10000
- 1 <= nums[i] < 2^31

**解题思路：**[链接](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-ii-lcof/solution/mian-shi-ti-56-ii-shu-zu-zhong-shu-zi-chu-xian-d-4/)

如下图所示，考虑数字的二进制形式，对于出现三次的数字，各 二进制位 出现的次数都是 3 的倍数。
因此，统计所有数字的各二进制位中 1 的出现次数，并对 3 求余，结果则为只出现一次的数字。

![Picture1.png](https://pic.leetcode-cn.com/28f2379be5beccb877c8f1586d8673a256594e0fc45422b03773b8d4c8418825-Picture1.png)

方法一：有限状态自动机

```java
class Solution {
    public int singleNumber(int[] nums) {
        int ones = 0, twos = 0;
        for(int num : nums){
            ones = ones ^ num & ~twos;
            twos = twos ^ num & ~ones;
        }
        return ones;
    }
}
```

**时间复杂度 ：** O(N) 其中 N 位数组 nums 的长度；遍历数组占用 O(N) ，每轮中的常数个位运算操作占用 O(32×3×2)=O(1) 。

**空间复杂度：** O(1)  变量 ones , twos 使用常数大小的额外空间。

方法二：遍历统计

> 此方法相对容易理解，但效率较低，总体推荐方法一。

实际上，只需要修改求余数值 m ，即可实现解决 **除了一个数字以外，其余数字都出现 m 次** 的通用问题。

```java
class Solution {
    public int singleNumber(int[] nums) {
        int[] counts = new int[32];//建立一个长度为 32 的数组，通过以上方法可记录所有数字的各二进制位的1的出现次数。
        for(int num : nums) {
            for(int j = 0; j < 32; j++) {
                counts[j] += num & 1;//更新第 j 位
                num >>>= 1;//第 j 位 --> 第 j + 1 位
            }
        }
        int res = 0, m = 3;
        for(int i = 0; i < 32; i++) {
            res <<= 1;
            res |= counts[31 - i] % m;// 得到 只出现一次的数字 的第 (31 - i) 位 
        }
        return res;
    }
}
```

**时间复杂度：**O(N)。其中 N位数组 nums 的长度；遍历数组占用 O(N)，每轮中的常数个位运算操作占用 O(1) 。

**空间复杂度：** O(1)。数组 counts长度恒为 32 ，占用常数大小的额外空间。

方法三：HashMap

```java
class Solution {
    // 使用 HashMap 记录各个数字出现的次数
    public int singleNumber(int[] nums) {
        Map<Integer, Integer> map = new HashMap();

        for(int i = nums.length - 1; i >= 0; --i){
            int key = nums[i];
            if(!map.containsKey(key)){
                // 如果之前没有遇到这一数字，则放入 map 中
                map.put(key, 1);
            }else{
                 // 如果之前遇到过这一数字，则出现次数加 1
                 map.put(key, map.get(key) + 1); 
                
            }
        }

        for(Map.Entry<Integer, Integer> entry: map.entrySet()){
            if(entry.getValue() == 1){
                return entry.getKey();
            }
        }

        return -1;
    }
}
```

- 时间复杂度 O(n) 
- 空间复杂度 O(n) 

### [003] 复杂链表的复制

请实现 copyRandomList 函数，复制一个复杂链表。在复杂链表中，每个节点除了有一个 next 指针指向下一个节点，还有一个 random 指针指向链表中的任意节点或者 null。

示例 1：

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/01/09/e1.png)

```
输入：head = [[7,null],[13,0],[11,4],[10,2],[1,0]]
输出：[[7,null],[13,0],[11,4],[10,2],[1,0]]
```

示例 2：

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/01/09/e2.png)

```
输入：head = [[1,1],[2,1]]
输出：[[1,1],[2,1]]
```

示例 3：

**![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/01/09/e3.png)**

```
输入：head = [[3,null],[3,0],[3,null]]
输出：[[3,null],[3,0],[3,null]]
```

示例 4：

```
输入：head = []
输出：[]
解释：给定的链表为空（空指针），因此返回 null。
```


提示：

- -10000 <= Node.val <= 10000
- Node.random 为空（null）或指向链表中的节点。
- 节点数目不超过 1000 。

---

本题链表的节点新增了 random 指针，指向链表中的 任意节点 或者 null 。这个 random 指针意味着在复制过程中，除了构建前驱节点和当前节点的引用指向 pre.next ，还要构建前驱节点和其随机节点的引用指向 pre.random 。

本题难点： 在复制链表的过程中构建新链表各节点的 random 引用指向。

![Picture1.png](https://pic.leetcode-cn.com/1604747285-ELUgCd-Picture1.png)

```java
class Solution {
    public Node copyRandomList(Node head) {
        Node cur = head;
        Node dum = new Node(0), pre = dum;
        while(cur != null) {
            Node node = new Node(cur.val); // 复制节点 cur
            pre.next = node;               // 新链表的 前驱节点 -> 当前节点
            // 新链表的 「 前驱节点 -> 当前节点 」 无法确定
            // pre.random = "???";         
            cur = cur.next;                // 遍历下一节点
            pre = node;                    // 保存当前新节点
        }
        return dum.next;
    }
}
```

> 本文介绍 「哈希表」 ，「拼接 + 拆分」 两种方法。哈希表方法比较直观；拼接 + 拆分方法的空间复杂度更低。

方法一：哈希表

利用哈希表的查询特点，考虑构建 **原链表节点** 和 **新链表对应节点** 的键值对映射关系，再遍历构建新链表各节点的 `next` 和 `random` 引用指向即可。

```java
class Solution {
    public Node copyRandomList(Node head) {
        //1. 若头节点 head 为空节点，直接返回 null
        if(head == null) return null;
        //2. 初始化：节点 cur 指向头节点, 哈希表
        Node cur = head;
        Map<Node, Node> map = new HashMap<>();
        // 3. 复制各节点，并建立 “原节点 -> 新节点” 的 Map 映射
        while(cur != null) {
            map.put(cur, new Node(cur.val));
            cur = cur.next;
        }
        cur = head;
        // 4. 构建新链表的 next 和 random 指向
        while(cur != null) {
            map.get(cur).next = map.get(cur.next);
            map.get(cur).random = map.get(cur.random);
            cur = cur.next;
        }
        // 5. 返回新链表的头节点
        return map.get(head);
    }
}
```

- **时间复杂度 O(N)：** 两轮遍历链表，使用 O(N) 时间。
- **空间复杂度 O(N)：** 哈希表 `dic` 使用线性大小的额外空间。

方法二：拼接 + 拆分

考虑构建`原节点 1 -> 新节点 1 -> 原节点 2 -> 新节点 2 -> …… `的拼接链表，如此便可在访问原节点的 random 指向节点的同时找到新对应新节点的 random 指向节点。

```java
class Solution {
    public Node copyRandomList(Node head) {
        if(head == null) return null;
        Node cur = head;
        // 1. 复制各节点，并构建拼接链表
        while(cur != null) {
            Node tmp = new Node(cur.val);
            tmp.next = cur.next;
            cur.next = tmp;
            cur = tmp.next;
        }
        // 2. 构建各新节点的 random 指向
        cur = head;
        while(cur != null) {
            if(cur.random != null)
                cur.next.random = cur.random.next;
            cur = cur.next.next;
        }
        // 3. 拆分两链表
        cur = head.next;
        Node pre = head, res = head.next;
        while(cur.next != null) {
            pre.next = pre.next.next;
            cur.next = cur.next.next;
            pre = pre.next;
            cur = cur.next;
        }
        pre.next = null; // 单独处理原链表尾节点
        return res;      // 返回新链表头节点
    }
}
```

- **时间复杂度 O(N)：** 三轮遍历链表，使用 O(N)时间。
- **空间复杂度 O(1) ：** 节点引用变量使用常数大小的额外空间。

### [004] 数组中数字出现的次数

一个整型数组 nums 里除两个数字之外，其他数字都出现了两次。请写程序找出这两个只出现一次的数字。要求时间复杂度是O(n)，空间复杂度是O(1)。

示例 1：

```
输入：nums = [4,1,4,6]
输出：[1,6] 或 [6,1]
```

示例 2：

```
输入：nums = [1,2,10,4,1,4,3,3]
输出：[2,10] 或 [10,2]
```


限制：

- 2 <= nums.length <= 10000

方法一：

先对所有数字进行一次异或，得到两个出现一次的数字的异或值。

在异或结果中找到任意为 1 的位。

根据这一位对所有的数字进行分组。

> 把所有数字分成两组，使得：
>
> 1. 两个只出现一次的数字在不同的组中；
>
> 2. 相同的数字会被分到相同的组中。
>
> 那么对两个组分别进行异或操作，即可得到答案的两个数字。这是解决这个问题的关键。
>
> 根据"任意为 1 的位"进行分组，这一位为1说明一个数字该位为1，另一个数字该位为0，可以确保2个数字分布在2组中。

在每个组内进行异或操作，得到两个数字。g

```java
class Solution {
    public int[] singleNumbers(int[] nums) {
        int ret = 0;
        for (int n : nums) {
            ret ^= n;
        }
        int div = 1;
        while ((div & ret) == 0) {
            div <<= 1;
        }
        int a = 0, b = 0;
        for (int n : nums) {
            if ((div & n) != 0) {
                a ^= n;
            } else {
                b ^= n;
            }
        }
        return new int[]{a, b};
    }
}
```

- 时间复杂度：O(n)，我们只需要遍历数组两次。
- 空间复杂度：O(1)，只需要常数的空间存放若干变量。

### [005] 重建二叉树

输入某二叉树的前序遍历和中序遍历的结果，请重建该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。

例如，给出

```java
前序遍历 preorder = [3,9,20,15,7]
中序遍历 inorder = [9,3,15,20,7]
```

返回如下的二叉树：

    	3
       / \
      9  20
        /  \
       15   7

限制：

- 0 <= 节点个数 <= 5000

方法一：

算法分析：

前序遍历性质： 节点按照 `[ 根节点 | 左子树 | 右子树 ]` 排序。
中序遍历性质： 节点按照 `[ 左子树 | 根节点 | 右子树 ]` 排序。

根据以上性质，可得出以下推论：

- 前序遍历的首元素 为 树的根节点 node 的值。
- 在中序遍历中搜索根节点 node 的索引 ，可将 中序遍历 划分为 `[ 左子树 | 根节点 | 右子树 ]` 
- 根据中序遍历中的左 / 右子树的节点数量，可将 前序遍历 划分为 `[ 根节点 | 左子树 | 右子树 ]` 

**递推参数：** 根节点在前序遍历的索引 root 、子树在中序遍历的左边界 left 、子树在中序遍历的右边界 right ；

**终止条件：** 当 left > right ，代表已经越过叶节点，此时返回 null ；

**递推工作：**

1. **建立根节点 `node` ：** 节点值为 `preorder[root]` ；
2. **划分左右子树：** 查找根节点在中序遍历 `inorder` 中的索引 `i` ；
3. **构建左右子树：** 开启左右子树递归；

|        | 根节点索引          | 中序遍历左边界 | 中序遍历右边界 |
| ------ | ------------------- | -------------- | -------------- |
| 左子树 | root + 1            | left           | i - 1          |
| 右子树 | root + i - left + 1 | i + 1          | right          |

> i - left + root + 1含义为 根节点索引 + 左子树长度 + 1

**返回值：** 回溯返回 node ，作为上一层递归中根节点的左 / 右子节点；

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
    int[] preorder;
    HashMap<Integer, Integer> dic = new HashMap<>();
    public TreeNode buildTree(int[] preorder, int[] inorder) {
        this.preorder = preorder;
        for(int i = 0; i < inorder.length; i++)
            dic.put(inorder[i], i);
        return recur(0,0,inorder.length-1);
    }

    public TreeNode recur(int root, int left, int right) {
        if(left > right) return null;
        int head_val = preorder[root];
        int head_index = dic.get(head_val);
        TreeNode head = new TreeNode(head_val);
        head.left = recur(root + 1, left, head_index - 1);
        head.right = recur(root + head_index - left + 1,head_index + 1,right);
        return head;
    }
}
```

> 注意：本文方法只适用于 “无重复节点值” 的二叉树。

- 时间复杂度 O(N) ： 其中 N 为树的节点数量。初始化 HashMap 需遍历 inorder ，占用 O(N)。递归共建立 N 个节点，每层递归中的节点建立、搜索操作占用 O(1)，因此使用 O(N) 时间。
- 空间复杂度 O(N) ： HashMap 使用 O(N) 额外空间。最差情况下，树退化为链表，递归深度达到 N ，占用 O(N) 额外空间；最好情况下，树为满二叉树，递归深度为logN ，占用 O(logN) 额外空间。



