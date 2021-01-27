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

**问题**： 循环必须使用 **while 或 for**，因此本方法不可取，直接排除。

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
    public int sumNums(int n) {
        boolean flag = n > 1 && (n += sumNums(n - 1)) > 0;
        return n;
    }
}
```

>Java 中，为构成语句，需加一个辅助布尔量 xx ，否则会报错；
>Java 中，开启递归函数需改写为 sumNums(n - 1) > 0 ，此整体作为一个布尔量输出，否则会报错；

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

在每个组内进行异或操作，得到两个数字。

```java
class Solution {
    public int[] singleNumbers(int[] nums) {
        //对所有数字进行一次异或，得到两个出现一次的数字的异或值
        int ret = 0;
        for (int n : nums) {
            ret ^= n;
        }
        int div = 1;
        //在异或结果中找到任意为 1 的位
        while ((div & ret) == 0) {
            div <<= 1;
        }
        int a = 0, b = 0;
        for (int n : nums) {
            //根据这一位对所有的数字进行分组
            //两个只出现一次的数字在不同的组中
            //对两个组分别进行异或操作
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

> root + i - left + 1含义为 根节点索引 + 左子树长度 + 1

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
    HashMap<Integer, Integer> dic = new HashMap<>();

    public TreeNode buildTree(int[] preorder, int[] inorder) {
        for(int i = 0; i < inorder.length; i++)
            dic.put(inorder[i], i);
        return recur(preorder,0,0,inorder.length-1);
    }

    public TreeNode recur(int[] preorder, int root, int left, int right) {
        if(left > right) return null;
        int head_val = preorder[root];
        int head_index = dic.get(head_val);
        TreeNode head = new TreeNode(head_val);
        head.left = recur(preorder,root + 1, left, head_index - 1);
        head.right = recur(preorder,root + head_index - left + 1,head_index + 1,right);
        return head;
    }
}
```

> 注意：本文方法只适用于 “无重复节点值” 的二叉树。

- 时间复杂度 O(N) ： 其中 N 为树的节点数量。初始化 HashMap 需遍历 inorder ，占用 O(N)。递归共建立 N 个节点，每层递归中的节点建立、搜索操作占用 O(1)，因此使用 O(N) 时间。
- 空间复杂度 O(N) ： HashMap 使用 O(N) 额外空间。最差情况下，树退化为链表，递归深度达到 N ，占用 O(N) 额外空间；最好情况下，树为满二叉树，递归深度为logN ，占用 O(logN) 额外空间。

方法二：

解题思路确定根节点的值，把根节点做出来，然后递归构造左右子树即可

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/gibkIz0MVqdF8ZItXTVByS26EcqBSS9W6zvlia07hHvYB5JTKLTHCAmDW9I8dX8c8LmSo1ibejUHGibgH6zhMXBCmw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

对于代码中的`rootVal`和`index`变量，就是下图这种情况：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/gibkIz0MVqdF8ZItXTVByS26EcqBSS9W6cuUtHIdXvXjbicaaZnpBWzEO1ZLfCGn9ntniaEicl5Et2wiarGaSq2GCZw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

对于左右子树对应的`inorder`数组的起始索引和终止索引比较容易确定：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/gibkIz0MVqdF8ZItXTVByS26EcqBSS9W6BFJp9KicjbvfTdvhU3vaDFEqaUiaNF1q3HzkyFjnpypG8XrGzJXdpeLg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

对于`preorder`数组，可以通过左子树的节点数推导出来，假设左子树的节点数为`leftSize = index - inStart;`，确定左右数组对应的起始索引和终止索引：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/gibkIz0MVqdF8ZItXTVByS26EcqBSS9W6Awr35eI0tibAJ2qW6pDUpgWTv5icgDhRhniaIJg3dpYib7Ph5kqDneL08A/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

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
    //用map保持索引，避免每次查找
    Map<Integer,Integer> map = new HashMap();
    public TreeNode buildTree(int[] preorder, int[] inorder){
        for (int i = 0; i < inorder.length; i++) {
            map.put(inorder[i],i);
        }
        return build(preorder, 0, preorder.length - 1,
                inorder, 0, inorder.length - 1);
    }

    TreeNode build(int[] preorder, int preStart, int preEnd,
                   int[] inorder, int inStart, int inEnd) {

        if (preStart > preEnd)  return null;
        
        // root 节点对应的值就是前序遍历数组的第一个元素
        int rootVal = preorder[preStart];
        // rootVal 在中序遍历数组中的索引
        int index = map.get(rootVal);
        //或使用如下代码不需要占用额外空间
//        for (int i = inStart; i <= inEnd; i++) {
//            if (inorder[i] == rootVal) {
//                index = i;
//                break;
//            }
//        }
        int leftSize = index - inStart;

        // 先构造出当前根节点
        TreeNode root = new TreeNode(rootVal);
        // 递归构造左右子树
        root.left = build(preorder, preStart + 1, preStart + leftSize,
                inorder, inStart, index - 1);

        root.right = build(preorder, preStart + leftSize + 1, preEnd,
                inorder, index + 1, inEnd);
        return root;
    }
}
```

- 时间复杂度：O(n)，其中 n 是树中的节点个数。

- 空间复杂度：O(n)，除去存储哈希映射 O(n) 空间之外，我们还需要使用 O(h)（其中 h 是树的高度）的空间存储栈。这里 h < n，所以（在最坏情况下）总空间复杂度为 O(n)。

### [006] 礼物的最大价值

在一个 m*n 的棋盘的每一格都放有一个礼物，每个礼物都有一定的价值（价值大于 0）。你可以从棋盘的左上角开始拿格子里的礼物，并每次向右或者向下移动一格、直到到达棋盘的右下角。给定一个棋盘及其上面的礼物的价值，请计算你最多能拿到多少价值的礼物？

示例 1:

```
输入: 
[
  [1,3,1],
  [1,5,1],
  [4,2,1]
]
输出: 12
解释: 路径 1→3→5→2→1 可以拿到最多价值的礼物
```


提示：

- 0 < grid.length <= 200
- 0 < grid[0].length <= 200

方法一：动态规划

根据题目说明，单元格只可能从上边单元格或左边单元格到达。

设` f(i, j)` 为从棋盘左上角走至单元格 (i ,j) 的礼物最大累计价值，易得到以下递推关系：`f(i,j)` 等于` f(i,j-1) `和 `f(i-1,j)` 中的较大值加上当前单元格礼物价值 `grid(i,j) `。
$$
f(i,j) = \max[f(i,j-1), f(i-1,j)] + grid(i,j)
$$
因此，可用动态规划解决此问题，以上公式便为转移方程。

![Picture1.png](https://pic.leetcode-cn.com/73153e75d74b1f48ac47244681caacc8ad20ca2ffd2dee2f70a2768dee09d073-Picture1.png)

**状态定义：** 设动态规划矩阵 dp ，`dp(i,j)` 代表从棋盘的左上角开始，到达单元格` (i,j) `时能拿到礼物的最大累计价值。

**转移方程：**

- 当 i = 0 且 j = 0 时，为起始元素；
- 当 i = 0 且 j != 0 时，为矩阵第一行元素，只可从左边到达；
- 当 i != 0 且 j = 0 时，为矩阵第一列元素，只可从上边到达；
- 当 i != 0 且 j != 0 时，可从左边或上边到达；

![img](https://pic.leetcode-cn.com/67cf85128a890bac4a7e38062f728ce536ebecd6c3595dd99e94f7f4cb2edd9f-Picture2.png)

初始状态： `dp[0][0] = grid[0][0]` ，即到达单元格 (0,0) 时能拿到礼物的最大累计价值为 `grid[0][0]` ；
返回值： `dp[m-1][n-1]` ，m, n 分别为矩阵的行高和列宽，即返回 dp 矩阵右下角元素。

**空间复杂度优化：**

由于 `dp[i][j] `只与 `dp[i-1][j]`, `dp[i][j-1]` , `grid[i][j]` 有关系，因此可以将原矩阵 grid 用作 dp 矩阵，即直接在 grid上修改即可。

应用此方法可省去 dp 矩阵使用的额外空间，因此空间复杂度从 O(MN) 降至 O(1)。

```java
class Solution {
    public int maxValue(int[][] grid) {
        int m = grid.length, n = grid[0].length;
        for(int i = 0; i < m; i++) {
            for(int j = 0; j < n; j++) {
                if(i == 0 && j == 0) continue;
                if(i == 0) grid[i][j] += grid[i][j - 1] ;
                else if(j == 0) grid[i][j] += grid[i - 1][j];
                else grid[i][j] += Math.max(grid[i][j - 1], grid[i - 1][j]);
            }
        }
        return grid[m - 1][n - 1];
    }
}
```

以上代码逻辑清晰，和转移方程直接对应，但仍可提升效率：当 grid 矩阵很大时， i = 0 或 j = 0 的情况仅占极少数，相当循环每轮都冗余了一次判断。因此，可先初始化矩阵第一行和第一列，再开始遍历递推。

```java
class Solution {
    public int maxValue(int[][] grid) {
        int m = grid.length, n = grid[0].length;
        for(int j = 1; j < n; j++) // 初始化第一行
            grid[0][j] += grid[0][j - 1];
        for(int i = 1; i < m; i++) // 初始化第一列
            grid[i][0] += grid[i - 1][0];
        for(int i = 1; i < m; i++)
            for(int j = 1; j < n; j++) 
                grid[i][j] += Math.max(grid[i][j - 1], grid[i - 1][j]);
        return grid[m - 1][n - 1];
    }
}
```

复杂度分析：

- 时间复杂度： O(MN)，M, N 分别为矩阵行高、列宽；动态规划需遍历整个 grid 矩阵，使用 O(MN) 时间。

- 空间复杂度：O(1)，原地修改使用常数大小的额外空间。

### [007] 从上到下打印二叉树

从上到下打印出二叉树的每个节点，同一层的节点按照从左到右的顺序打印。

例如:

给定二叉树: [3,9,20,null,null,15,7],

    	3
       / \
      9  20
        /  \
       15   7

返回：

`[3,9,20,15,7]`


提示：

- 节点总数 <= 1000

方法一：BFS

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
    public int[] levelOrder(TreeNode root) {
        if(root == null) return new int[0];
        List<Integer> ans = new ArrayList<>();
        Queue<TreeNode> queue = new LinkedList<>();
        queue.add(root);
        while(!queue.isEmpty()) {
            TreeNode node = queue.poll();
            ans.add(node.val);
            if(node.left != null) queue.add(node.left);
            if(node.right != null) queue.add(node.right);
        }
        int[] res = new int[ans.size()];
        for(int i = 0; i < ans.size(); i++)
            res[i] = ans.get(i);
        
        return res;
    }
}
```

- 时间复杂度 O(N) ： N 为二叉树的节点数量，即 BFS 需循环 N 次。

- 空间复杂度 O(N) ： 最差情况下，即当树为平衡二叉树时，最多有 N/2 个树节点同时在 queue 中，使用 O(N) 大小的额外空间。

### [008] 丑数

我们把只包含质因子 2、3 和 5 的数称作丑数（Ugly Number）。求按从小到大的顺序的第 n 个丑数。

示例:

```
输入: n = 10
输出: 12
解释: 1, 2, 3, 4, 5, 6, 8, 9, 10, 12 是前 10 个丑数。
```

说明:  

- 1 是丑数。
- n 不超过1690。

方法一：动态规划

> 丑数的递推性质： 丑数只包含因子 2, 3, 5 ，因此有 “丑数 == 某较小丑数 * 某因子” （例如：10 = 5×2）。

![Picture1.png](https://pic.leetcode-cn.com/837411664f096417badf857fa51e77fd30cb1309a5637c37d24d8a4a48a42b03-Picture1.png)

**状态定义：** 设动态规划列表 dp ，dp[i] 代表第 i + 1 个丑数。

**转移方程：**

1. 当索引 a, b, c 满足以下条件时， dp[i] 为三种情况的最小值；

2. 每轮计算 dp[i] 后，需要更新索引 a, b, c 的值，使其始终满足方程条件。实现方法：分别独立判断 dp[i] 和 `dp[a]×2` , `dp[b]×3` , `dp[c]×5` 的大小关系，若相等则将对应索引 a , b , c 加 1 。

```
dp[a]×2>dp[i−1]≥dp[a−1]×2
dp[b]×3>dp[i−1]≥dp[b−1]×3
dp[c]×5>dp[i−1]≥dp[c−1]×5

dp[i]=min(dp[a]×2,dp[b]×3,dp[c]×5)
```

- **初始状态：** dp[0] = 1，即第一个丑数为 1；
- **返回值：** dp[n-1] ，即返回第 n 个丑数。

```java
class Solution {
    public int nthUglyNumber(int n) {
        //所有的丑数都是 较小丑数 * 某因子（2，3，5）
        //用三个指针，分别指向2，3，5较小丑数的索引
        int a = 0, b = 0, c = 0;
        int[] dp = new int[n];
        //初始化1为丑数
        dp[0] = 1;
        for(int i = 1; i < n; i++) {
            //计算当前较小丑数索引对应的新一批丑数
            int n2 = dp[a] * 2, n3 = dp[b] * 3, n5 = dp[c] * 5;
            //设置当前索引丑数值为新一批中最小的值
            dp[i] = Math.min(Math.min(n2, n3), n5);
            //最新的丑数索引加1
            if(dp[i] == n2) a++;
            if(dp[i] == n3) b++;
            if(dp[i] == n5) c++;
        }
        return dp[n - 1];
    }
}
```

复杂度分析：

- 时间复杂度 O(N)： 其中 N = n ，动态规划需遍历计算 dp 列表。
- 空间复杂度 O(N) ： 长度为 N 的 dp 列表使用 O(N)) 的额外空间。

### [009] 二叉搜索树与双向链表

输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的循环双向链表。要求不能创建任何新的节点，只能调整树中节点指针的指向。

为了让您更好地理解问题，以下面的二叉搜索树为例：

 ![img](https://assets.leetcode.com/uploads/2018/10/12/bstdlloriginalbst.png)

我们希望将这个二叉搜索树转化为双向循环链表。链表中的每个节点都有一个前驱和后继指针。对于双向循环链表，第一个节点的前驱是最后一个节点，最后一个节点的后继是第一个节点。

下图展示了上面的二叉搜索树转化成的链表。“head” 表示指向链表中有最小元素的节点。

![img](https://assets.leetcode.com/uploads/2018/10/12/bstdllreturndll.png)

 

特别地，我们希望可以就地完成转换操作。当转化完成以后，树中节点的左指针需要指向前驱，树中节点的右指针需要指向后继。还需要返回链表中的第一个节点的指针。

方法一：

本文解法基于性质：二叉搜索树的中序遍历为 **递增序列** 。

将 二叉搜索树 转换成一个 “排序的循环双向链表” ，其中包含三个要素：

- 排序链表： 节点应从小到大排序，因此应使用中序遍历 “从小到大”访问树的节点；
- 双向链表： 在构建相邻节点（设前驱节点 pre ，当前节点 cur ）关系时，不仅应 pre.right = cur ，也应 cur.left = pre。
- 循环链表： 设链表头节点 headhead 和尾节点 tail ，则应构建 head.left = tail 和 tail.right = head 。

![Picture14.png](https://pic.leetcode-cn.com/963f2da36712b57f870a5e81d839a03737a347f19bab268cf1fd6fd60649711e-Picture14.png)



```java
/*
// Definition for a Node.
class Node {
    public int val;
    public Node left;
    public Node right;

    public Node() {}

    public Node(int _val) {
        val = _val;
    }

    public Node(int _val,Node _left,Node _right) {
        val = _val;
        left = _left;
        right = _right;
    }
};
*/
class Solution {
    Node head, pre;
    public Node treeToDoublyList(Node root) {
        if(root==null) return null;
        //可以把right理解为后继，left理解为前驱
        dfs(root);
        //进行头节点和尾节点的相互指向，顺序可以颠倒
        pre.right = head;
        head.left =pre;
        return head;
    }

    public void dfs(Node cur){
        if(cur==null) return;
        dfs(cur.left);
        //pre用于记录双向链表中位于cur左侧的节点，即上一次迭代中的cur
        //当pre==null时，cur左侧没有节点,即此时cur为双向链表中的头节点
        if(pre==null) head = cur;
        //反之，pre!=null时，cur左侧存在节点pre，需要进行pre.right=cur的操作。
        else pre.right = cur;     
        cur.left = pre;//pre是否为null对这句没有影响,且这句放在上面两句if else之前也是可以的。
        pre = cur;//pre指向当前的cur
        dfs(cur.right);//全部迭代完成后，pre指向双向链表中的尾节点
    }
}
```

- 时间复杂度 O(N) ： N 为二叉树的节点数，中序遍历需要访问所有节点。
- 空间复杂度 O(N) ： 最差情况下，即树退化为链表时，递归深度达到 N，系统使用 O(N) 栈空间。

### [010] 股票的最大利润

假设把某股票的价格按照时间先后顺序存储在数组中，请问买卖该股票一次可能获得的最大利润是多少？

示例 1:

```
输入: [7,1,5,3,6,4]
输出: 5
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
     注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格。
```

示例 2:

```
输入: [7,6,4,3,1]
输出: 0
解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。
```


限制：

0 <= 数组长度 <= 10^5

方法一：动态规划

**状态定义：** 设动态规划列表 dp ，dp[i] 代表以 prices[i] 为结尾的子数组的最大利润（以下简称为 前 i 日的最大利润 ）

**转移方程：** 由于题目限定 “买卖该股票一次” ，因此前 i 日最大利润 dp[i] 等于前 i - 1 日最大利润 dp[i-1] 和第 i 日卖出的最大利润中的最大值。
$$
前i日最大利润=max(前(i−1)日最大利润,第i日价格−前i日最低价格)
$$

$$
dp[i]=max(dp[i−1],prices[i]−min(prices[0:i]))
$$

优化后的状态转移方程：
$$
profit=max(profit,prices[i]−min(cost,prices[i])
$$
**初始状态**： dp[0] = 0 ，即首日利润为 0 ；

**返回值**： dp[n - 1] ，其中 n 为 dp 列表长度。

![Picture1.png](https://pic.leetcode-cn.com/4880911383c41712612103c612e390f1ee271e4eb921f22476836dc46aa3a58a-Picture1.png)



```java
class Solution {
    public int maxProfit(int[] prices) {
        int cost = Integer.MAX_VALUE, profit = 0;
        for(int price : prices) {
            cost = Math.min(cost, price);
            profit = Math.max(profit, price - cost);
        }
        return profit;
    }
}
```

- 时间复杂度 O(N) ： 其中 N 为 prices 列表长度，动态规划需遍历 prices 。
- 空间复杂度 O(1)： 变量 cost 和 profit 使用常数大小的额外空间。

### [011] 栈的压入、弹出序列

输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如，序列 {1,2,3,4,5} 是某栈的压栈序列，序列 {4,5,3,2,1} 是该压栈序列对应的一个弹出序列，但 {4,3,5,1,2} 就不可能是该压栈序列的弹出序列。 

示例 1：

```
输入：pushed = [1,2,3,4,5], popped = [4,5,3,2,1]
输出：true
解释：我们可以按以下顺序执行：
push(1), push(2), push(3), push(4), pop() -> 4,
push(5), pop() -> 5, pop() -> 3, pop() -> 2, pop() -> 1
```

示例 2：

```
输入：pushed = [1,2,3,4,5], popped = [4,3,5,1,2]
输出：false
解释：1 不能在 2 之前弹出。
```


提示：

- 0 <= pushed.length == popped.length <= 1000
- 0 <= pushed[i], popped[i] < 1000
- pushed 是 popped 的排列。

方法一：辅助栈

借用一个辅助栈 stack ，模拟 压入 / 弹出操作的排列。根据是否模拟成功，即可得到结果。

入栈操作： 按照压栈序列的顺序执行。

出栈操作： 每次入栈后，循环判断 “栈顶元素 == 弹出序列的当前元素” 是否成立，将符合弹出序列顺序的栈顶元素全部弹出。

> 由于题目规定 栈的所有数字均不相等 ，因此在循环入栈中，每个元素出栈的位置的可能性是唯一的（若有重复数字，则具有多个可出栈的位置）。因而，在遇到 “栈顶元素 == 弹出序列的当前元素” 就应立即执行出栈。

算法流程：

- **初始化**： 辅助栈 stack，弹出序列的索引 i 

- **遍历压栈序列**： 各元素记为 num 
  - 元素 num 入栈
  - 循环出栈：若 stack 的栈顶元素 == 弹出序列元素 popped[i] ，则执行出栈与 i++

- **返回值**： 若 stack 为空，则此弹出序列合法

```java
class Solution {
    public boolean validateStackSequences(int[] pushed, int[] popped) {
        Stack<Integer> stack = new Stack<>();
        int i = 0;
        for(int num : pushed) {
            stack.push(num); // num 入栈
            while(!stack.isEmpty() && stack.peek() == popped[i]) { // 循环判断与出栈
                stack.pop();
                i++;
            }
        }
        return stack.isEmpty();
    }
}
```

- 时间复杂度 O(N) ： 其中 N 为列表 pushed 的长度；每个元素最多入栈与出栈一次，即最多共 2N 次出入栈操作。
- 空间复杂度 O(N) ： 辅助栈 stack 最多同时存储 N 个元素。

方法二：辅助数组

原理同上

```java
class Solution {
    public boolean validateStackSequences(int[] pushed, int[] popped) {
        int x = 0;
        int[] t = new int[pushed.length];

        for (int i = 0, j = 0; i < pushed.length; i++, x++) {
            t[x] = pushed[i];//模拟入栈
            while (x >= 0 && j < popped.length && t[x] == popped[j]) {// 循环判断与模拟出栈
                j++;
                x--;
            }
        }
        return x == 0;
    }
}
```

- 时间复杂度 O(N)
- 空间复杂度 O(N) 

### [012] 构建乘积数组

给定一个数组 `A[0,1,…,n-1]`，请构建一个数组 `B[0,1,…,n-1]`，其中 B 中的元素 `B[i]=A[0]×A[1]×…×A[i-1]×A[i+1]×…×A[n-1]`。不能使用除法。

 示例:

```
输入: [1,2,3,4,5]
输出: [120,60,40,30,24]
```


提示：

- 所有元素乘积之和不会溢出 32 位整数
- a.length <= 100000

方法一：

难点在于 不能使用除法 ，即需要 只用乘法 生成数组 BB 。根据题目对 B[i]B[i] 的定义，可列表格，如下图所示。

根据表格的主对角线（全为 1 ），可将表格分为 上三角 和 下三角 两部分。分别迭代计算下三角和上三角两部分的乘积，即可 不使用除法 就获得结果。

![Picture1.png](https://pic.leetcode-cn.com/6056c7a5009cb7a4674aab28505e598c502a7f7c60c45b9f19a8a64f31304745-Picture1.png)

算法流程：

- 初始化：数组 B ，其中 B[0] = 1；辅助变量 tmp = 1；
- 计算 B[i] 的 下三角 各元素的乘积，直接乘入 B[i] ；
- 计算 B[i] 的 上三角 各元素的乘积，记为 tmp ，并乘入 B[i] ；
- 返回 B 。

基础实现：

```java
class Solution {
    public int[] constructArr(int[] a) {
        if(a == null || a.length == 0) return new int[0];
        int len = a.length;
        int[] left = new int[len];
        int[] right = new int[len];
        left[0] = right[len - 1] = 1;

        //下三角
        for (int i = 1; i < len; i++) {
            left[i] = left[i - 1] * a[i - 1];
        }
        //上三角
        for (int i = len - 2; i >= 0; i--) {
            right[i] = right[i + 1] * a[i + 1];
        }
		//求乘积
        int[] ans = new int[len];
        for (int i = 0; i < len; i++) {
            ans[i] = left[i] * right[i];
        }
        return ans;
    }
}
```

- 时间复杂度 O(N) ，遍历了3次数组，最后一次求乘积可放在上三角的遍历中，可优化为2次。
- 空间复杂度 O(N) ，使用了2个额外数组，可直接在返回数组中操作，有优化为O(1)。

**优化**

```java
class Solution {
    public int[] constructArr(int[] a) {
        if(a == null || a.length == 0) return new int[0];
        int[] b = new int[a.length];
        b[0] = 1;
        int tmp = 1;
        //计算下三角
        for(int i = 1; i < a.length; i++) {
            b[i] = b[i - 1] * a[i - 1];
        }
        //计算上三角
        for(int i = a.length - 2; i >= 0; i--) {
            tmp *= a[i + 1];
            b[i] *= tmp;
        }
        return b;
    }
}
```

- 时间复杂度 O(N) ： 其中 N 为数组长度，两轮遍历数组 a ，使用 O(N) 时间。
- 空间复杂度 O(1)： 变量 tmp 使用常数大小额外空间（数组 b 作为返回值，不计入复杂度考虑）。

### [013] 从上到下打印二叉树 III

请实现一个函数按照之字形顺序打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右到左的顺序打印，第三行再按照从左到右的顺序打印，其他行以此类推。

例如:

给定二叉树: [3,9,20,null,null,15,7],

    	3
       / \
      9  20
        /  \
       15   7

返回其层次遍历结果：

```
[
  [3],
  [20,9],
  [15,7]
]
```


提示：

- 节点总数 <= 1000

方法一：层序遍历 + 双端队列

注意：本题额外要求 **打印顺序交替变化**

利用双端队列的两端皆可添加元素的特性，设打印列表（双端队列） `tmp` ，并规定：

- 奇数层 则添加至 `tmp` **尾部** ，
- 偶数层 则添加至 `tmp` **头部** 。

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
    public List<List<Integer>> levelOrder(TreeNode root) {
        Queue<TreeNode> queue = new LinkedList<>();
        List<List<Integer>> res = new ArrayList<>();
        if(root != null) queue.add(root);
        while(!queue.isEmpty()) {
            LinkedList<Integer> tmp = new LinkedList<>();
            for(int i = queue.size(); i > 0; i--) {
                TreeNode node = queue.poll();
                if(res.size() % 2 == 0) tmp.addLast(node.val); // 偶数层 -> 队列头部
                else tmp.addFirst(node.val); // 奇数层 -> 队列尾部
                if(node.left != null) queue.add(node.left);
                if(node.right != null) queue.add(node.right);
            }
            res.add(tmp);
        }
        return res;
    }
}
```

- 时间复杂度 O(N) ： N 为二叉树的节点数量，即 BFS 需循环 N 次，占用 O(N)；双端队列的队首和队尾的添加和删除操作的时间复杂度均为 O(1)。
- 空间复杂度 O(N) ： 最差情况下，即当树为满二叉树时，最多有 N/2 个树节点 同时 在 deque 中，使用 O(N) 大小的额外空间。

方法二：层序遍历 + 双端队列（奇偶层逻辑分离）

> 方法一代码简短、容易实现；但需要判断每个节点的所在层奇偶性，即冗余了 N*N* 次判断。通过将奇偶层逻辑拆分，可以消除冗余的判断。

**BFS 循环**： 循环打印奇 / 偶数层，当 deque 为空时跳出；

- 打印奇数层： 从左向右 打印，先左后右 加入下层节点；
- 若 deque 为空，说明向下无偶数层，则跳出；
- 打印偶数层： 从右向左 打印，先右后左 加入下层节点；

```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        Deque<TreeNode> deque = new LinkedList<>();
        List<List<Integer>> res = new ArrayList<>();
        if(root != null) deque.add(root);
        while(!deque.isEmpty()) {
            // 打印奇数层
            List<Integer> tmp = new ArrayList<>();
            for(int i = deque.size(); i > 0; i--) {
                // 从左向右打印
                TreeNode node = deque.removeFirst();
                tmp.add(node.val);
                // 先左后右加入下层节点
                if(node.left != null) deque.addLast(node.left);
                if(node.right != null) deque.addLast(node.right);
            }
            res.add(tmp);
            if(deque.isEmpty()) break; // 若为空则提前跳出
            // 打印偶数层
            tmp = new ArrayList<>();
            for(int i = deque.size(); i > 0; i--) {
                // 从右向左打印
                TreeNode node = deque.removeLast();
                tmp.add(node.val);
                // 先右后左加入下层节点
                if(node.right != null) deque.addFirst(node.right);
                if(node.left != null) deque.addFirst(node.left);
            }
            res.add(tmp);
        }
        return res;
    }
}
```

- 时间复杂度 O(N)：同方法一。
- 空间复杂度 O(N)：同方法一。

方法三：层序遍历 + 倒序

> 此方法的优点是只用列表即可，无需其他数据结构。
>
> 偶数层倒序：若 `res` 的长度为 **奇数** ，说明当前是偶数层，则对 `tmp` 执行 **倒序** 操作。

```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        Queue<TreeNode> queue = new LinkedList<>();
        List<List<Integer>> res = new ArrayList<>();
        if(root != null) queue.add(root);
        while(!queue.isEmpty()) {
            List<Integer> tmp = new ArrayList<>();
            for(int i = queue.size(); i > 0; i--) {
                TreeNode node = queue.poll();
                tmp.add(node.val);
                if(node.left != null) queue.add(node.left);
                if(node.right != null) queue.add(node.right);
            }
            if(res.size() % 2 == 1) Collections.reverse(tmp);
            res.add(tmp);
        }
        return res;
    }
}
```

时间复杂度 O(N)： N 为二叉树的节点数量，即 BFS 需循环 N 次，占用 O(N) 。共完成 少于 N 个节点的倒序操作，占用 O(N) 。
空间复杂度 O(N)： 最差情况下，即当树为满二叉树时，最多有 N/2 个树节点同时在 queue 中，使用 O(N) 大小的额外空间。

### [014] 二叉树中和为某一值的路径

输入一棵二叉树和一个整数，打印出二叉树中节点值的和为输入整数的所有路径。从树的根节点开始往下一直到叶节点所经过的节点形成一条路径。 

示例:
给定如下二叉树，以及目标和 sum = 22，

              5
             / \
            4   8
           /   / \
          11  13  4
         /  \    / \
        7    2  5   1
返回:

```
[
   [5,4,11,2],
   [5,8,4,5]
]
```


提示：

- 节点总数 <= 10000

方法一：

> 本问题是典型的二叉树方案搜索问题，使用回溯法解决，其包含 **先序遍历 + 路径记录** 两部分。

**先序遍历**： 按照 “根、左、右” 的顺序，遍历树的所有节点。

**路径记录**： 在先序遍历中，记录从根节点到当前节点的路径。当路径为满足根节点到叶节点形成的路径且各节点值的和等于目标值 sum 时，将此路径加入结果列表。

**递推工作**：

- 路径更新： 将当前节点值 root.val 加入路径 path ；
- 目标值更新： tar = tar - root.val（即目标值 tar 从 sum 减至 00 ）；
- 路径记录： 当 ① root 为叶节点 且 ② 路径和等于目标值 ，则将此路径 path 加入 res 。
- 先序遍历： 递归左 / 右子节点。
- 路径恢复： 向上回溯前，需要将当前节点从路径 path 中删除，即执行 path.pop() 。

```java
class Solution {
    LinkedList<List<Integer>> res = new LinkedList<>();
    LinkedList<Integer> path = new LinkedList<>();
    public List<List<Integer>> pathSum(TreeNode root, int sum) {
        dfs(root, sum);
        return res;
    }

    public void dfs(TreeNode root, int tar) {
        if (root == null) return;
        path.add(root.val);
        tar -= root.val;
        if (tar == 0 && root.left == null && root.right == null)
            res.add(new LinkedList(path));
        dfs(root.left, tar);
        dfs(root.right, tar);
        //向上回溯前，需要将当前节点从路径 path 中删除
        path.removeLast();
    }
}
```

- 时间复杂度 O(N)： N 为二叉树的节点数，先序遍历需要遍历所有节点。

- 空间复杂度 O(N)： 最差情况下，即树退化为链表时，path 存储所有树节点，使用 O(N) 额外空间。

### [015] 把数组排成最小的数

输入一个非负整数数组，把数组里所有数字拼接起来排成一个数，打印能拼接出的所有数字中最小的一个。

示例 1:

```
输入: [10,2]
输出: "102"
```

示例 2:

```
输入: [3,30,34,5,9]
输出: "3033459"
```


提示:

0 < nums.length <= 100

说明:

- 输出结果可能非常大，所以你需要返回一个字符串而不是整数
- 拼接起来的数字可能会有前导 0，最后结果不需要去掉前导 0

方法一：

此题求拼接起来的 “最小数字” ，本质上是一个排序问题。
排序判断规则： 设 nums 任意两数字的字符串格式 x 和 y ，则

- 若拼接字符串 x + y > y + x ，则 x > y ；
- 反之，若 x + y < y + x，则 x < y ；

根据以上规则，套用任何排序方法对 nums 执行排序即可。

![Picture1.png](https://pic.leetcode-cn.com/5f7afd0b198405c178c41e1f60a2b54037f2a931a3df6a4056bc908c902aa567-Picture1.png)

**算法流程**：

- 初始化： 字符串列表 strs ，保存各数字的字符串格式；
- 列表排序： 应用以上 “排序判断规则” ，对 strs 执行排序；
- 返回值： 拼接 strs 中的所有字符串，并返回。

快速排序：需修改快速排序函数中的排序判断规则。

```java
class Solution {
    public String minNumber(int[] nums) {
        String[] strs = new String[nums.length];
        for(int i = 0; i < nums.length; i++)
            strs[i] = String.valueOf(nums[i]);
        fastSort(strs, 0, strs.length - 1);
        StringBuilder res = new StringBuilder();
        for(String s : strs)
            res.append(s);
        return res.toString();
    }
    void fastSort(String[] strs, int l, int r) {
        if(l >= r) return;
        int i = l, j = r;
        String tmp = strs[i];
        while(i < j) {
            while((strs[j] + strs[l]).compareTo(strs[l] + strs[j]) >= 0 && i < j) j--;
            while((strs[i] + strs[l]).compareTo(strs[l] + strs[i]) <= 0 && i < j) i++;
            tmp = strs[i];
            strs[i] = strs[j];
            strs[j] = tmp;
        }
        strs[i] = strs[l];
        strs[l] = tmp;
        fastSort(strs, l, i - 1);
        fastSort(strs, i + 1, r);
    }
}
```

复杂度分析：

- 时间复杂度 O(NlogN) ： N 为最终返回值的字符数量（ strs 列表的长度 ≤N ）；使用快排或内置函数的平均时间复杂度为 O(NlogN) ，最差为 O(N^2)
- 空间复杂度 O(N) ： 字符串列表 strs 占用线性大小的额外空间。

方法二：内置函数

原理同上

```java
class Solution {
    public String minNumber(int[] nums) {
        String[] strs = new String[nums.length];
        for(int i = 0; i < nums.length; i++) 
            strs[i] = String.valueOf(nums[i]);
        Arrays.sort(strs, (x, y) -> (x + y).compareTo(y + x));
        StringBuilder res = new StringBuilder();
        for(String s : strs)
            res.append(s);
        return res.toString();
    }
}
```

- 时间复杂度 O(NlogN)

- 空间复杂度 O(N) 

### [016] 剪绳子※

给你一根长度为 n 的绳子，请把绳子剪成整数长度的 m 段（m、n都是整数，n>1并且m>1），每段绳子的长度记为 `k[0],k[1]...k[m-1]` 。请问 `k[0]*k[1]*...*k[m-1]` 可能的最大乘积是多少？例如，当绳子的长度是8时，我们把它剪成长度分别为2、3、3的三段，此时得到的最大乘积是18。

示例 1：

```
输入: 2
输出: 1
解释: 2 = 1 + 1, 1 × 1 = 1
```

示例 2:

```
输入: 10
输出: 36
解释: 10 = 3 + 3 + 4, 3 × 3 × 4 = 36
```

提示：

- 2 <= n <= 58

[官方题解](https://leetcode-cn.com/problems/integer-break/solution/zheng-shu-chai-fen-by-leetcode-solution/)

方法一：动态规划

dp定义：`dp[i] `表示将正整数 i 拆分成至少两个正整数的和之后，这些正整数的最大乘积。

边界条件：`dp[1] = dp[2] = 1`，表示长度为 2 的绳子最大乘积为 1；

当 `i≥2` 时，假设对正整数 i 拆分出的第一个正整数是` j（1≤j<i）`，则有以下两种方案：

- 将 i 拆分成 j 和 i-j 的和，且 i-j 不再拆分成多个正整数，此时的乘积是`j×(i−j)`；

- 将 i 拆分成 j 和 i-j 的和，且 i-j 继续拆分成多个正整数，此时的乘积是`j×dp[i−j]`。

因此，当 j 固定时，有 `dp[i]=max(j×(i−j),j×dp[i−j])`。由于 j 的取值范围是 1 到 i-1，需要遍历所有的 j 得到 dp[i] 的最大值，可以这样理解：

![14.jpg](https://pic.leetcode-cn.com/82b25ac6bcb742f31e5202e4af993d98abfea6a0c385379b214440bbb84b9bb4-14.jpg)

状态转移方程：`dp[i] = max(dp[i], max((i - j) * j, j * dp[i - j])) 1<=j<=i`。

```java
class Solution {
    public int integerBreak(int n) {
        int[] dp = new int[n + 1];
        for (int i = 2; i <= n; i++) {
            int curMax = 0;
            for (int j = 1; j < i; j++) {
                curMax = Math.max(curMax, Math.max(j * (i - j), j * dp[i - j]));
            }
            dp[i] = curMax;
        }
        return dp[n];
    }
}
```

时间复杂度：O(n^2)

空间复杂度：O(n)

方法二：优化的动态规划

```java
class Solution {
    public int integerBreak(int n) {
        if (n < 4) {
            return n - 1;
        }
        int[] dp = new int[n + 1];
        dp[2] = 1;
        for (int i = 3; i <= n; i++) {
            dp[i] = Math.max(Math.max(2 * (i - 2), 2 * dp[i - 2]), Math.max(3 * (i - 3), 3 * dp[i - 3]));
        }
        return dp[n];
    }
}
```

时间复杂度：O(n)

空间复杂度：O(n)

方法三：数学

```java
class Solution {
    public int integerBreak(int n) {
        if (n <= 3) {
            return n - 1;
        }
        int quotient = n / 3;
        int remainder = n % 3;
        if (remainder == 0) {
            return (int) Math.pow(3, quotient);
        } else if (remainder == 1) {
            return (int) Math.pow(3, quotient - 1) * 4;
        } else {
            return (int) Math.pow(3, quotient) * 2;
        }
    }
}
```

- 时间复杂度：O(1)。涉及到的操作包括计算商和余数，以及幂次运算，时间复杂度都是常数。

- 空间复杂度：O(1)。只需要使用常数复杂度的额外空间。

### [017] 字符串的排列

输入一个字符串，打印出该字符串中字符的所有排列。 

你可以以任意顺序返回这个字符串数组，但里面不能有重复元素。

示例:

```
输入：s = "abc"
输出：["abc","acb","bac","bca","cab","cba"]
```

限制：

- 1 <= s 的长度 <= 8

方法一：

排列方案数量： 对于一个长度为 n 的字符串（假设字符互不重复），其排列共有 `n×(n−1)×(n−2)…×2×1` 种方案。

![Picture1.png](https://pic.leetcode-cn.com/dc4659dbda6d54f50a8c897647fb7c52e2b8200e741c4d6e25306dfe51f93bb6-Picture1.png)

重复方案与剪枝： 当字符串存在重复字符时，排列方案中也存在重复方案。为排除重复方案，需在固定某位字符时，保证 “每种字符只在此位固定一次” ，即遇到重复字符时不交换，直接跳过。从 DFS 角度看，此操作称为 “剪枝” 。

![Picture2.png](https://pic.leetcode-cn.com/edbbe4db611791ca63e582e8b0c754261e8d7464edace38420ce3087eb96d9a5-Picture2.png)

递归解析：

- **终止条件**： 当 `x = len(c) - 1` 时，代表所有位已固定（最后一位只有 1 种情况），则将当前组合 c 转化为字符串并加入 res，并返回；

- **递推参数**： 当前固定位 x ；

- **递推工作**： 初始化一个 Set ，用于排除重复的字符；将第 x 位字符与 `i∈[x,len(c)]` 字符分别交换，并进入下层递归；
  - 剪枝： 若 `c[i]` 在 Set 中，代表其是重复字符，因此“剪枝”；
  - 将 `c[i]` 加入 Set ，以便之后遇到重复字符时剪枝；
  - 固定字符： 将字符 `c[i]` 和 `c[x]` 交换，即固定 `c[i]` 为当前位字符；
  - 开启下层递归： 调用 `dfs(x + 1)` ，即开始固定第 x + 1 个字符；
  - 还原交换： 将字符 `c[i]` 和 `c[x]` 交换（还原之前的交换）；



```java
class Solution {
    List<String> res = new LinkedList<>();
    char[] c;
    public String[] permutation(String s) {
        c = s.toCharArray();
        dfs(0);
        return res.toArray(new String[res.size()]);
    }
    void dfs(int x) {
        if(x == c.length - 1) {
            res.add(String.valueOf(c)); // 添加排列方案
            return;
        }
        HashSet<Character> set = new HashSet<>();
        for(int i = x; i < c.length; i++) {
            if(set.contains(c[i])) continue; // 重复，因此剪枝
            set.add(c[i]);
            //交换元素，这里很是巧妙，当在第二层dfs(1),x = 1,那么i = 1或者 2， 不是交换1和1，要就是交换1和2
            swap(i, x); // 交换，将 c[i] 固定在第 x 位 
            dfs(x + 1); // 开启固定第 x + 1 位字符
            //返回时交换回来，这样保证到达第1层的时候，一直都是abc。
            //这里捋顺一下，开始一直都是abc，那么第一位置总共就3个交换
            //分别是a与a交换，这个就相当于 x = 0, i = 0;
            //     a与b交换            x = 0, i = 1;
            //     a与c交换            x = 0, i = 2;
            //第一个元素固定后，每个引出两条路径,
            //     b与b交换            x = 1, i = 1;
            //     b与c交换            x = 1, i = 2;
            swap(i, x); // 恢复交换
        }
    }
    void swap(int a, int b) {
        char tmp = c[a];
        c[a] = c[b];
        c[b] = tmp;
    }
}
```

时间复杂度 O(N!)  

空间复杂度 O(N^2) ： 全排列的递归深度为 N ，系统累计使用栈空间大小为 O(N) ；递归中辅助 Set 累计存储的字符数量最多为 N + (N-1) + ... + 2 + 1 = (N+1)N/2，即占用 O(N^2)的额外空间。

### [018] n个骰子的点数

把n个骰子扔在地上，所有骰子朝上一面的点数之和为s。输入n，打印出s的所有可能的值出现的概率。

你需要用一个浮点数数组返回答案，其中第 i 个元素代表这 n 个骰子所能掷出的点数集合中第 i 小的那个的概率。

示例 1:

```
输入: 1
输出: [0.16667,0.16667,0.16667,0.16667,0.16667,0.16667]
```

示例 2:

```
输入: 2
输出: [0.02778,0.05556,0.08333,0.11111,0.13889,0.16667,0.13889,0.11111,0.08333,0.05556,0.02778]
```


限制：

- 1 <= n <= 11

方法一：动态规划

根据概率的计算公式，点数 k 出现概率就算公式为：

```
P(k) = k出现的次数 / 总次数
```

投掷 n 个骰子，所有点数出现的总次数是 `6^n`，因为一共有 n 枚骰子，每枚骰子的点数都有 6 种可能出现的情况。

题目目的就是 **计算出投掷完 n 枚骰子后每个点数出现的次数**。

状态表示：`dp[i][j]`，表示投掷完 i 枚骰子后，点数 j 的出现次数。

- 首先用数组的第一维来表示阶段，也就是投掷完了几枚骰子。
- 然后用第二维来表示投掷完这些骰子后，可能出现的点数。
- 数组的值就表示，该阶段各个点数出现的次数。

> 注意，这里的点数 j 指的是前 n 枚骰子的点数和，而不是第 n 枚骰子的点数

状态转移方程：

单看第 n 枚骰子，它的点数可能为 `1, 2, 3, ... , 6`，因此投掷完 n 枚骰子后点数 j 出现的次数，可以由投掷完 `n-1` 枚骰子后，对应点数` j-1, j-2, j-3, ... , j-6` 出现的次数之和转化过来。

```
for (第n枚骰子的点数 i = 1; i <= 6; i ++) {
    dp[n][j] += dp[n-1][j - i]
}
```

边界处理：第一阶段的状态：投掷完 1 枚骰子后，它的可能点数分别为 `1, 2, 3, ... , 6`，并且每个点数出现的次数都是 1

```java
class Solution {
    public double[] dicesProbability(int n) {
        int[][] dp = new int[n + 1][6 * n + 1];
        for(int i = 1; i <= 6; i++)
            dp[1][i] = 1;
        
        for(int i = 2; i <= n; i++)
            for(int j = i; j <= 6 * i; j++)
                for(int k = 1; k <= 6 && k <= j; k++)
                    dp[i][j] += dp[i-1][j - k];
        
        double[] ans = new double[6 * n - n + 1];
        for(int i = n; i <= 6 * n; i++)
            ans[i - n] = ((double)dp[n][i]) / (Math.pow(6,n));
        return ans;
    }
}
```

空间复杂度：O(n^2)

时间复杂度：O(n^2)

空间优化：每个阶段的状态都只和它前一阶段的状态有关，因此我们不需要用额外的一维来保存所有阶段。

```java
class Solution {
    public double[] dicesProbability(int n) {
        double[] res = new double[5*n+1];
        //定义状态矩阵
        int[] dp = new int[6*n];
        //边界处理
        for (int i = 0; i < 6; i++) {
            dp[i]=1;
        }
        //状态矩阵计算
        for (int i = 1; i < n; i++) {
            //第i行表示的是i+1个骰子的情况数，从i到6（i+1)-1。
            //比如第0行表示的是1个骰子的情况，从0到5。
            for (int j = 6*(i+1)-1; j >i-1 ; j--) {
                //状态转移
                dp[j]=0;
                for (int k = 1; k <7 ; k++) {
                    //第i+1个骰子只能使用第i个骰子的结果，第i个最小值为i在第i-1位置记录。
                    if(j-k<i-1)
                        break;
                    dp[j] += dp[j-k];
                }
            }
        }
        double all = Math.pow(6,n);
        for (int i = n-1, j = 0; i <6*n ; i++,j++) {
            res[j]=dp[i]/all;
        }
        return  res;

    }
}
```

空间复杂度：O(n^2)

时间复杂度：O(n)

### [019] 把数字翻译成字符串

给定一个数字，我们按照如下规则把它翻译为字符串：0 翻译成 “a” ，1 翻译成 “b”，……，11 翻译成 “l”，……，25 翻译成 “z”。一个数字可能有多个翻译。请编程实现一个函数，用来计算一个数字有多少种不同的翻译方法。

示例 1:

```
输入: 12258
输出: 5
解释: 12258有5种不同的翻译，分别是"bccfi", "bwfi", "bczi", "mcfi"和"mzi"
```


提示：

- 0 <= num < 2^31

方法一：

翻译的规则，字符串的第 i 位置：

- 可以单独作为一位来翻译
- 如果第 i - 1 位和第 i 位组成的数字在 10 到 25 之间，可以把这两位连起来翻译

> 记数字 num 第 i 位数字为 x_i，数字 num 的位数为 n；

![Picture1.png](https://pic.leetcode-cn.com/e231fde16304948251633cfc65d04396f117239ea2d13896b1d2678de9067b42-Picture1.png)

状态定义：设动态规划列表 dp ，dp[i]代表以 x_i 为结尾的数字的翻译方案数量

转移方程：若 x_i和 x_{i-1} 组成的两位数字可以被翻译，则 `dp[i] = dp[i - 1] + dp[i - 2]`；否则 `dp[i] = dp[i - 1]`。

初始状态：`dp[0] = dp[1] = 1` ，即 “无数字” 和 “第 1 位数字” 的翻译方法数量均为 1 ；

返回值： dp[n]，即此数字的翻译方案数量。

优化空间复杂度： 由于 dp[i] 只与 dp[i - 1] 有关，因此可使用两个变量 a, b 分别记录 dp[i], dp[i - 1]，两变量交替前进即可。此方法可省去 dp 列表使用的 O(N) 的额外空间。

```java
class Solution {
    public int translateNum(int num) {
        String s = String.valueOf(num);
        int a = 1, b = 1;
        for(int i = 2; i <= s.length(); i++) {
            String tmp = s.substring(i - 2, i);
            int c = tmp.compareTo("10") >= 0 && tmp.compareTo("25") <= 0 ? a + b : a;
            b = a;
            a = c;
        }
        return a;
    }
}
```

- 时间复杂度为 O(N)，*N* 为字符串 s 的长度（即数字 num 的位数 log(num)），其决定了循环次数。

- 空间复杂度为 O(N)，这里用了滚动数组，动态规划部分的空间代价是 O(1) 的，但是这里用了一个临时变量把数字转化成了字符串，故空间复杂度也是N。

### [020] 二叉搜索树的后序遍历序列

输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历结果。如果是则返回 true，否则返回 false。假设输入的数组的任意两个数字都互不相同。

参考以下这颗二叉搜索树：

         5
        / \
       2   6
      / \
     1   3

示例 1：

```
输入: [1,6,3,2,5]
输出: false
```

示例 2：

```
输入: [1,3,2,6,5]
输出: true
```


提示：

- 数组长度 <= 1000

方法一：递归分治

后序遍历定义：` [ 左子树 | 右子树 | 根节点 ] `，即遍历顺序为 “左、右、根” 。

二叉搜索树定义： 左子树中所有节点的值 < 根节点的值；右子树中所有节点的值 > 根节点的值；其左、右子树也分别为二叉搜索树。

根据二叉搜索树的定义，可以通过递归，判断所有子树的 **正确性** （即其后序遍历是否满足二叉搜索树的定义） ，若所有子树都正确，则此序列为二叉搜索树的后序遍历。

```java
class Solution {
    public boolean verifyPostorder(int[] postorder) {
        return recur(postorder, 0, postorder.length - 1);
    }
    boolean recur(int[] postorder, int i, int j) {
        if(i >= j-1) return true;//任意两个数都可以构成搜索树的后序遍历
        int p = i;
        while(postorder[p] < postorder[j]) p++;
        int m = p;
        while(postorder[p] > postorder[j]) p++;
        return p == j && recur(postorder, i, m - 1) && recur(postorder, m, j - 1);
    }
}
```

- 时间复杂度 O(N^2)： 每次调用 `recur(i,j) `减去一个根节点，因此递归占用 O(N) ；最差情况下（即当树退化为链表），每轮递归都需遍历树所有节点，占用 O(N) 。
- 空间复杂度 O(N) ： 最差情况下（即当树退化为链表），递归深度将达到 N 。

方法二：辅助单调栈

后序遍历倒序： `[ 根节点 | 右子树 | 左子树 ]` 。类似 先序遍历的镜像 ，即先序遍历为 “根、左、右” 的顺序，而后序遍历的倒序为 “根、右、左” 顺序。

[参考](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-hou-xu-bian-li-xu-lie-lcof/solution/mian-shi-ti-33-er-cha-sou-suo-shu-de-hou-xu-bian-6/)

算法流程：

1. 初始化： 单调栈 stack，父节点值 root=正无穷（初始值为正无穷大，可把树的根节点看为此无穷大节点的左孩子）；
2. 倒序遍历 postorder ：记每个节点为 r_i
   - 判断： 若 r_i>root ，说明此后序遍历序列不满足二叉搜索树定义，直接返回 false
   - 更新父节点 root ： 当栈不为空 且 r_i<stack.peek() 时，循环执行出栈，并将出栈节点赋给 root
   - 入栈： 将当前节点 r_i入栈
3. 若遍历完成，则说明后序遍历满足二叉搜索树定义，返回 true 。

```java
class Solution { 
    public boolean verifyPostorder(int[] postorder) {
        Stack<Integer> stack = new Stack<>();
        int root = Integer.MAX_VALUE;
        for(int i = postorder.length - 1; i >= 0; i--) {
            if(postorder[i] > root) return false;
            while(!stack.isEmpty() && stack.peek() > postorder[i])
            	root = stack.pop();
            stack.add(postorder[i]);
        }
        return true;
    }
}
```

- 时间复杂度 O(N)： 遍历 postorder 所有节点，各节点均入栈 / 出栈一次，使用 O(N) 时间。
- 空间复杂度 O(N)： 最差情况下，单调栈 stackstack 存储所有节点，使用 O(N) 额外空间。

### [021] 机器人的运动范围

地上有一个m行n列的方格，从坐标 [0,0] 到坐标 [m-1,n-1] 。一个机器人从坐标 [0, 0] 的格子开始移动，它每次可以向左、右、上、下移动一格（不能移动到方格外），也不能进入行坐标和列坐标的数位之和大于k的格子。例如，当k为18时，机器人能够进入方格 [35, 37] ，因为3+5+3+7=18。但它不能进入方格 [35, 38]，因为3+5+3+8=19。请问该机器人能够到达多少个格子？

示例 1：

```
输入：m = 2, n = 3, k = 1
输出：3
```

示例 2：

```
输入：m = 3, n = 1, k = 0
输出：1
```

提示：

- 1 <= n,m <= 100
- 0 <= k <= 20

方法一：广度优先搜索

将行坐标和列坐标数位之和大于 `k` 的格子看作障碍物，那么这道题就是一道传统的搜索题目。

搜索的过程中搜索方向可以缩减为向右和向下，而不必再向上和向左进行搜索。

```java
class Solution {
    public int movingCount(int m, int n, int k) {
        if (k == 0) {
            return 1;
        }
        Queue<int[]> queue = new LinkedList<int[]>();
        // 向右和向下的方向数组
        int[] dx = {0, 1};
        int[] dy = {1, 0};
        //记录是否已经访问
        boolean[][] vis = new boolean[m][n];
        //队列中放入
        queue.offer(new int[]{0, 0});
        //置true，表示已经访问
        vis[0][0] = true;
        int ans = 1;
        while (!queue.isEmpty()) {
            int[] cell = queue.poll();
            int x = cell[0], y = cell[1];
            //向右和向下check
            for (int i = 0; i < 2; ++i) {
                int tx = dx[i] + x;
                int ty = dy[i] + y;
                //越界/访问过/大于k
                if (tx < 0 || tx >= m || ty < 0 || ty >= n || vis[tx][ty] || get(tx) + get(ty) > k) {
                    continue;
                }
                //放入队列
                queue.offer(new int[]{tx, ty});
                //置已经访问过
                vis[tx][ty] = true;
                ans++;
            }
        }
        return ans;
    }
    
    private int get(int x) {
        int res = 0;
        while (x != 0) {
            res += x % 10;
            x /= 10;
        }
        return res;
    }

}
```

- 时间复杂度：O(mn)，其中 m 为方格的行数，n 为方格的列数。考虑所有格子都能进入，那么搜索的时候一个格子最多会被访问的次数为常数，所以时间复杂度为 O(2mn)=O(mn)。

- 空间复杂度：O(mn)，其中 m 为方格的行数，n 为方格的列数。搜索的时候需要一个大小为 O(mn) 的标记结构用来标记每个格子是否被走过。

方法二：递推

方法一提到搜索的方向只需要朝下或朝右，可以得出一种递推的求解方法。

定义 `vis[i][j]` 为 `(i, j)`坐标是否可达，如果可达返回 1，否则返回 0。

首先 `(i, j)` 本身需要可以进入，因此需要先判断 i 和 j 的数位之和是否大于 k ，如果大于的话直接设置 `vis[i][j]` 为不可达即可。

否则，前面提到搜索方向只需朝下或朝右，因此 `(i, j)` 的格子只会从 `(i - 1, j)` 或者 `(i, j - 1)` 两个格子走过来（不考虑边界条件），那么 `vis[i][j]` 是否可达的状态则可由如下公式计算得到：

`vis[i][j]=vis[i−1][j] or vis[i][j−1]`

即只要有一个格子可达，那么 `(i, j)` 这个格子就是可达的，因此我们只要遍历所有格子，递推计算出它们是否可达然后用变量 ans 记录可达的格子数量即可。

初始条件 `vis[i][j] = 1` ，递推计算的过程中注意边界的处理。

```java
class Solution {
    public int movingCount(int m, int n, int k) {
        if (k == 0) {
            return 1;
        }
        //记录是否可达
        boolean[][] vis = new boolean[m][n];
        int ans = 1;
        vis[0][0] = true;
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                //不符合
                if ((i == 0 && j == 0) || get(i) + get(j) > k) {
                    continue;
                }
                // 可达性判断
                // 搜索方向只需朝下或朝右，因此 (i, j) 的格子只会从 (i - 1, j) 或者 (i, j - 1) 两个格子走过来
                // 边界判断
                if (i - 1 >= 0) {
                    vis[i][j] |= vis[i - 1][j];
                }
                if (j - 1 >= 0) {
                    vis[i][j] |= vis[i][j - 1];
                }
                ans += vis[i][j] ? 1 : 0;
            }
        }
        return ans;
    }

    private int get(int x) {
        int res = 0;
        while (x != 0) {
            res += x % 10;
            x /= 10;
        }
        return res;
    }
}
```

- 时间复杂度：O(mn)，其中 m 为方格的行数， n 为方格的列数。一共有 O(mn) 个状态需要计算，每个状态递推计算的时间复杂度为 O(1)，所以总时间复杂度为 O(mn)。
- 空间复杂度：O(mn)，其中 m 为方格的行数，n 为方格的列数。我们需要 O(mn) 大小的结构来记录每个位置是否可达。

方法三：深度优先遍历 

暴力法模拟机器人在矩阵中的所有路径。DFS 通过递归，先朝一个方向搜到底，再回溯至上个节点，沿另一个方向搜索，以此类推。

在搜索中，遇到数位和超出目标值、此元素已访问，则应立即返回，称之为 `可行性剪枝` 。

**递归参数**： 当前元素在矩阵中的行列索引 i 和 j ，两者的数位和 si, sj 。

**终止条件**： 当 ① 行列索引越界 或 ② 数位和超出目标值 k 或 ③ 当前元素已访问过 时，返回 0 ，代表不计入可达解。

**递推工作**：

- **标记当前单元格** ：将索引 (i, j) 存入 visited 中，代表此单元格已被访问过。

- **搜索下一单元格**： 计算当前元素的 下、右 两个方向元素的数位和，并开启下层递归 。

**回溯返回值**： 返回 `1 + 右方搜索的可达解总数 + 下方搜索的可达解总数`，代表从本单元格递归搜索的可达解总数。

```java
class Solution {
    int m, n, k;
    boolean[][] visited;
    public int movingCount(int m, int n, int k) {
        this.m = m; this.n = n; this.k = k;
        this.visited = new boolean[m][n];
        return dfs(0, 0, 0, 0);
    }
    public int dfs(int i, int j, int si, int sj) {
        if(i >= m || j >= n || k < si + sj || visited[i][j]) return 0;
        visited[i][j] = true;
        return 1 + dfs(i + 1, j, get(i+1), sj)
                + dfs(i, j + 1, si, get(j+1));
    }

    private int get(int x) {
        int res = 0;
        while (x != 0) {
            res += x % 10;
            x /= 10;
        }
        return res;
    }
}
```

时间复杂度 O(mn) ： 最差情况下，机器人遍历矩阵所有单元格，此时时间复杂度为 O(mn) 。

空间复杂度 O(mn) ： 最差情况下，Set visited 内存储矩阵所有单元格的索引，使用 O(mn)的额外空间。

### [022] 队列的最大值

请定义一个队列并实现函数 max_value 得到队列里的最大值，要求函数max_value、push_back 和 pop_front 的**均摊**时间复杂度都是O(1)。

若队列为空，pop_front 和 max_value 需要返回 -1

示例 1：

```
输入: 
["MaxQueue","push_back","push_back","max_value","pop_front","max_value"]
[[],[1],[2],[],[],[]]
输出: [null,null,null,2,1,2]
```

示例 2：

```
输入: 
["MaxQueue","pop_front","max_value"]
[[],[],[]]
输出: [null,-1,-1]
```


限制：

- 1 <= push_back,pop_front,max_value的总操作数 <= 10000
- 1 <= value <= 10^5

方法一：维护一个单调的双端队列

本算法基于问题的一个重要性质：当一个元素进入队列的时候，它前面所有比它小的元素就不会再对答案产生影响。

可以设计这样的方法：从队列尾部插入元素时，我们可以提前取出队列中所有比这个元素小的元素，使得队列中只保留对结果有影响的数字。这样的方法等价于要求维持队列单调递减，即要保证每个元素的前面都没有比它小的元素。

高效实现一个始终递减的队列：只需要在插入每一个元素 value 时，从队列尾部依次取出比当前元素 value 小的元素，直到遇到一个比当前元素大或相等的元素 value' 即可。

上面的过程保证了只要在元素 value 被插入之前队列递减，那么在 value 被插入之后队列依然递减。

上面的过程需要从队列尾部取出元素，因此需要使用双端队列来实现。另外我们也需要一个辅助队列来记录所有被插入的值，以确定 pop_front 函数的返回值。

保证了队列单调递减后，求最大值时只需要直接取双端队列中的第一项即可。

```java
class MaxQueue {
    Queue<Integer> q;//辅助队列来记录所有被插入的值
    Deque<Integer> d;//双端队列记录最大的值

    public MaxQueue() {
        q = new LinkedList<Integer>();
        d = new LinkedList<Integer>();
    }
    
    public int max_value() {
        if (d.isEmpty()) {
            return -1;
        }
        return d.peekFirst();
    }
    
    public void push_back(int value) {
        //从队列尾部依次取出比当前元素 value 小的元素
        while (!d.isEmpty() && d.peekLast() < value) {
            d.pollLast();
        }
        d.offerLast(value);
        q.offer(value);
    }
    
    public int pop_front() {
        if (q.isEmpty()) {
            return -1;
        }
        int ans = q.poll();
        if (ans == d.peekFirst()) {
            d.pollFirst();
        }
        return ans;
    }
}


/**
 * Your MaxQueue object will be instantiated and called as such:
 * MaxQueue obj = new MaxQueue();
 * int param_1 = obj.max_value();
 * obj.push_back(value);
 * int param_3 = obj.pop_front();
 */
```

- 时间复杂度：O(1)（插入，删除，求最大值）
  删除操作于求最大值操作显然只需要 O(1) 的时间。
  而插入操作虽然看起来有循环，做一个插入操作时最多可能会有 n 次出队操作。但要注意，由于每个数字只会出队一次，因此对于所有的 n 个数字的插入过程，对应的所有出队操作也不会大于 n 次。因此将出队的时间均摊到每个插入操作上，时间复杂度为 O(1)。
- 空间复杂度：O(n)，需要用队列存储所有插入的元素。

### [023] 树的子结构

输入两棵二叉树A和B，判断B是不是A的子结构。(约定空树不是任意一个树的子结构)

B是A的子结构， 即 A中有出现和B相同的结构和节点值。

例如:
给定的树 A:

         3
        / \
       4   5
      / \
     1   2

给定的树 B：

```
   4 
  /
 1
```

返回 true，因为 B 与 A 的一个子树拥有相同的结构和节点值。

示例 1：

```
输入：A = [1,2,3], B = [3,1]
输出：false
```

示例 2：

```
输入：A = [3,4,5,1,2], B = [4,1]
输出：true
```

限制：

- 0 <= 节点个数 <= 10000

方法一：

若树 B 是树 A 的子结构，则子结构的根节点可能为树 A 的任意一个节点。因此，判断树 B 是否是树 A 的子结构，需完成以下两步工作：

- 先序遍历树 A 中的每个节点 n_A；（对应函数 `isSubStructure(A, B)`）
- 判断树 A 中 以 n_A为根节点的子树 是否包含树 B 。（对应函数 `recur(A, B)`）

![Picture1.png](https://pic.leetcode-cn.com/27d9f65b79ae4982fb58835d468c2a23ec2ac399ba5f38138f49538537264d03-Picture1.png)

**recur(A, B)** 函数：

终止条件：

- 当节点 B 为空：说明树 B 已匹配完成（越过叶子节点），因此返回 true ；
- 当节点 A 为空：说明已经越过树 A 叶子节点，即匹配失败，返回 false ；
- 当节点 A 和 B 的值不同：说明匹配失败，返回 false ；

返回值：

- 判断 A 和 B 的左子节点是否相等，即 recur(A.left, B.left) ；
- 判断 A 和 B 的右子节点是否相等，即 recur(A.right, B.right) ；

**isSubStructure(A, B)** 函数：

特例处理： 当 树 A 为空 或 树 B 为空 时，直接返回 false ；

返回值： 若树 B 是树 A 的子结构，则必满足以下三种情况之一，因此用或 || 连接；

- 以 节点 A 为根节点的子树 包含树 B ，对应 recur(A, B)；
- 树 B 是 树 A 左子树 的子结构，对应 isSubStructure(A.left, B)；
- 树 B 是 树 A 右子树 的子结构，对应 isSubStructure(A.right, B)；

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
    //遍历A树的节点，判断节点对于的子树是否包含B树
    public boolean isSubStructure(TreeNode A, TreeNode B) {
        return (A != null && B != null)
                && (recur(A, B)                 //A包含B
                || isSubStructure(A.left, B)    //A的左子树包含B
                || isSubStructure(A.right, B)); //A的右子树包含B
    }

    //判断以A树包含B
    boolean recur(TreeNode A, TreeNode B) {
        if(B == null) return true;
        if(A == null || A.val != B.val) return false;
        return recur(A.left, B.left) && recur(A.right, B.right);
    }
}
```

- 时间复杂度 O(MN) ： 其中 M,N 分别为树 A 和 树 B 的节点数量；先序遍历树 A 占用 O(M) ，每次调用 recur(A, B) 判断占用 O(N)。
- 空间复杂度 O(M) ： 当树 A 和树 B 都退化为链表时，递归调用深度最大。当 M≤N 时，遍历树 A 与递归判断的总递归深度为 M ；当 M>N 时，最差情况为遍历至树 A 叶子节点，此时总递归深度为 M。

### [024] 最长不含重复字符的子字符串

请从字符串中找出一个最长的不包含重复字符的子字符串，计算该最长子字符串的长度。

示例 1:

```
输入: "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
```

示例 2:

```
输入: "bbbbb"
输出: 1
解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。
```

示例 3:

```
输入: "pwwkew"
输出: 3
解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。
     请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。
```


提示：

- s.length <= 40000

方法一：滑动窗口+HashMap

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        //使用哈希表map统计各字符最后一次出现的索引位置
        Map<Character, Integer> dic = new HashMap<>();
        // 左指针 i, 最大值 res
        int i = -1, res = 0;
        for(int j = 0; j < s.length(); j++) {
            char c = s.charAt(j);
            if(dic.containsKey(c))//如果存在当前字符c
                i = Math.max(i, dic.get(c)); // 更新左指针 i
            dic.put(c, j); // 哈希表记录
            res = Math.max(res, j - i); // 更新结果，j代表右指针
        }
        return res;
    }
}
```

- 时间复杂度 O(N) ： 其中 N 为字符串长度
- 空间复杂度 O(1) ： 字符的 ASCII 码范围为0 ~ 127 ，哈希表 dic 最多使用 O(128) = O(1) 大小的额外空间。

### [025] 矩阵中的路径

请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。路径可以从矩阵中的任意一格开始，每一步可以在矩阵中向左、右、上、下移动一格。如果一条路径经过了矩阵的某一格，那么该路径不能再次进入该格子。例如，在下面的3×4的矩阵中包含一条字符串“bfce”的路径。

```
[["a","b","c","e"],
["s","f","c","s"],
["a","d","e","e"]]
```

但矩阵中不包含字符串“abfb”的路径，因为字符串的第一个字符b占据了矩阵中的第一行第二个格子之后，路径不能再次进入这个格子。 

示例 1：

```
输入：board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]], word = "ABCCED"
输出：true
```

示例 2：

```
输入：board = [["a","b"],["c","d"]], word = "abcd"
输出：false
```

提示：

- 1 <= board.length <= 200
- 1 <= board[i].length <= 200

方法一：DFS + 剪枝

DFS 解析：

- 递归参数： 当前元素在矩阵 board 中的行列索引 i 和 j ，当前目标字符在 word 中的索引 k 。
- 终止条件：
  返回 false ： (1) 行或列索引越界 或 (2) 当前矩阵元素与目标字符不同 或 (3) 当前矩阵元素已访问过 （ (3) 可合并至 (2) ） 。
  返回 true ： `k = len(word) - 1` ，即字符串 word 已全部匹配。
- 递推工作：
  标记当前矩阵元素： 将 `board[i][j]` 修改为 空字符` '' `，代表此元素已访问过，防止之后搜索时重复访问。
  搜索下一单元格： 朝当前元素的 上、下、左、右 四个方向开启下层递归，使用 `或` 连接 （代表只需找到一条可行路径就直接返回，不再做后续 DFS ），并记录结果至 res 。
  还原当前矩阵元素： 将 `board[i][j] `元素还原至初始值，即 `word[k]` 。
- 返回值： 返回布尔量 res ，代表是否搜索到目标字符串。

> 使用空字符（ Java: '\0' ）做标记是为了防止标记字符与矩阵原有字符重复。当存在重复时，此算法会将矩阵原有字符认作标记字符，从而出现错误

![Picture0.png](https://pic.leetcode-cn.com/1604944042-glmqJO-Picture0.png)

```java
class Solution {
    public boolean exist(char[][] board, String word) {
        char[] words = word.toCharArray();
        for(int i = 0; i < board.length; i++) {
            for(int j = 0; j < board[0].length; j++) {
                if(dfs(board, words, i, j, 0)) return true;
            }
        }
        return false;
    }
    boolean dfs(char[][] board, char[] word, int i, int j, int k) {
        if(i >= board.length || i < 0 || j < 0
           || j >= board[0].length
           || board[i][j] != word[k]) return false;
        if(k == word.length - 1) return true;
        board[i][j] = '\0';//临时赋值成‘/’，用来标记已访问的元素
        boolean res = dfs(board, word, i + 1, j, k + 1) 
            || dfs(board, word, i - 1, j, k + 1) 
            || dfs(board, word, i, j + 1, k + 1) 
            || dfs(board, word, i , j - 1, k + 1);
        board[i][j] = word[k];
        return res;
    }
}
```

> *M*,*N* 分别为矩阵行列大小， K 为字符串 `word` 长度。

- 时间复杂度 O(3^KMN)：最差情况下，需要遍历矩阵中长度为 K 字符串的所有方案，时间复杂度为 O(3^K)；矩阵中共有 MN 个起点，时间复杂度为 O(MN) 。

- 空间复杂度 O(K) ： 搜索过程中的递归深度不超过 K ，因此系统因函数调用累计使用的栈空间占用 O(K)（因为函数返回后，系统调用的栈空间会释放）。最坏情况下 K = MN，递归深度为 MN，此时系统栈使用 O(MN)的额外空间。

### [026] 二维数组中的查找

在一个 n * m 的二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个高效的函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

 示例:

现有矩阵 matrix 如下：

```
[
  [1,   4,  7, 11, 15],
  [2,   5,  8, 12, 19],
  [3,   6,  9, 16, 22],
  [10, 13, 14, 17, 24],
  [18, 21, 23, 26, 30]
]
```

给定 target = 5，返回 true。

给定 target = 20，返回 false。

限制：

- 0 <= n <= 1000

- 0 <= m <= 1000

方法一：暴力

```java
class Solution {
    public boolean findNumberIn2DArray(int[][] matrix, int target) {
        if (matrix == null || matrix.length == 0 || matrix[0].length == 0) {
            return false;
        }
        int rows = matrix.length, columns = matrix[0].length;
        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < columns; j++) {
                if (matrix[i][j] == target) {
                    return true;
                }
            }
        }
        return false;
    }
}
```

时间复杂度：O(nm)。二维数组中的每个元素都被遍历，因此时间复杂度为二维数组的大小。
空间复杂度：O(1)。

方法二：线性查找

算法流程：

从矩阵 matrix 左下角元素（索引设为 (i, j) ）开始遍历，并与目标值对比：

- 当 `matrix[i][j] > target` 时，执行 i-- ，即消去第 i 行元素；
- 当 `matrix[i][j] < target` 时，执行 j++ ，即消去第 j 列元素；
- 当 `matrix[i][j] = target` 时，返回 true，代表找到目标值。

若行索引或列索引越界，则代表矩阵中无目标值，返回 false。

> 右上角同理

```java
class Solution {
    public boolean findNumberIn2DArray(int[][] matrix, int target) {
        int i = matrix.length - 1, j = 0;
        while(i >= 0 && j < matrix[0].length)
        {
            //消去第 i 行元素
            if(matrix[i][j] > target) i--;
            //消去第 j 列元素
            else if(matrix[i][j] < target) j++;
            else return true;
        }
        return false;
    }
}

```

时间复杂度：O(n+m)。访问到的下标的行最多增加 n 次，列最多减少 m 次，因此循环体最多执行 n + m 次。
空间复杂度：O(1)。

### [027] 数字序列中某一位的数字※

数字以0123456789101112131415…的格式序列化到一个字符序列中。在这个序列中，第5位（从下标0开始计数）是5，第13位是1，第19位是4，等等。

请写一个函数，求任意第n位对应的数字。

示例 1：

```
输入：n = 3
输出：3
```

示例 2：

```
输入：n = 11
输出：0
```


限制：

- 0 <= n < 2^31

方法一：

[分析](https://leetcode-cn.com/problems/shu-zi-xu-lie-zhong-mou-yi-wei-de-shu-zi-lcof/solution/mian-shi-ti-44-shu-zi-xu-lie-zhong-mou-yi-wei-de-6/)

- 将 101112⋯ 中的每一位称为 数位 ，记为 n ；
- 将 10,11,12,⋯ 称为 数字 ，记为 num ；
- 数字 1010 是一个两位数，称此数字的 位数 为 2 ，记为 digit ；
- 每 digit 位数的起始数字（即：1,10,100,⋯），记为 start 。

![Picture1.png](https://pic.leetcode-cn.com/2cd7d8a6a881b697a43f153d6c10e0e991817d78f92b9201b6ab71e44cb619de-Picture1.png)

可推出各 digit 下的数位数量 count 的计算公式：

```
count=9×start×digit
```

根据以上分析，可将求解分为三步：

1. 确定 n 所在 数字 的 位数 ，记为 digit ；
2. 确定 n 所在的 数字 ，记为 num ；
3. 确定 n 是 num 中的哪一数位，并返回结果。

**第一步**：

![Picture2.png](https://pic.leetcode-cn.com/16836ca609f8b4d9af776b35eab4a4c4a86d76f4628a1bc931e56d197617bbb4-Picture2.png)

**第二步**：

![Picture3.png](https://pic.leetcode-cn.com/1f2cefd22a9825eb4a52d606a4aee2f93dd659d1b332d3b6a6ed68e5289e8d01-Picture3.png)

**第三步**：

![Picture4.png](https://pic.leetcode-cn.com/09af6bd37d9c79d9b904bedef01f0464aee1cd15e18d8a2ea86b70b312a830c3-Picture4.png)

代码实现：

```java
class Solution {
    public int findNthDigit(int n) {
        int digit = 1;
        long start = 1;
        long count = 9;
        while (n > count) { // 1.确定 n 所在 数字 的 位数 ，记为 digit
            n -= count;
            digit += 1;
            start *= 10;
            count = digit * start * 9;
        }
        long num = start + (n - 1) / digit; // 2.确定 n 所在的 数字 ，记为 num
        return Long.toString(num).charAt((n - 1) % digit) - '0'; // 3.确定 n 是 num 中的哪一数位，并返回结果
    }
}
```

时间复杂度 O(logn)

空间复杂度 O(logn)

### [028] 数值的整数次方※

实现函数`double Power(double base, int exponent)`，求base的exponent次方。不得使用库函数，同时不需要考虑大数问题。

示例 1:

```
输入: 2.00000, 10
输出: 1024.00000
```

示例 2:

```
输入: 2.10000, 3
输出: 9.26100
```

示例 3:

```
输入: 2.00000, -2
输出: 0.25000
解释: 2-2 = 1/22 = 1/4 = 0.25
```


说明:

- -100.0 < x < 100.0
- n 是 32 位有符号整数，其数值范围是 [−231, 231 − 1] 。

方法一：快速幂法

> 最简单的方法是通过循环将 n 个 x 乘起来，依次求 x^1, x^2, ..., x^{n-1}, x^nx，时间复杂度为 O(n)。
> 快速幂法可将时间复杂度降低至 O(logn) 

[分析](https://leetcode-cn.com/problems/shu-zhi-de-zheng-shu-ci-fang-lcof/solution/mian-shi-ti-16-shu-zhi-de-zheng-shu-ci-fang-kuai-s/)

```java
class Solution {
    public double myPow(double x, int n) {
        if(x == 0) return 0;
        long b = n;
        double res = 1.0;
        if(b < 0) {
            x = 1 / x;
            b = -b;
        }
        while(b > 0) {
            if((b & 1) == 1) res *= x;
            x *= x;
            b >>= 1;
        }
        return res;
    }
}
```

时间复杂度 O(logn) ： 二分的时间复杂度为对数级别。
空间复杂度 O(1) ： 占用常数大小额外空间。

### [029] 剪绳子 II※

给你一根长度为 n 的绳子，请把绳子剪成整数长度的 m 段（m、n都是整数，n>1并且m>1），每段绳子的长度记为 `k[0],k[1]...k[m - 1]` 。请问 `k[0]*k[1]*...*k[m - 1]` 可能的最大乘积是多少？例如，当绳子的长度是8时，我们把它剪成长度分别为2、3、3的三段，此时得到的最大乘积是18。

答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

示例 1：

```
输入: 2
输出: 1
解释: 2 = 1 + 1, 1 × 1 = 1
```

示例 2:

```
输入: 10
输出: 36
解释: 10 = 3 + 3 + 4, 3 × 3 × 4 = 36
```


提示：

- 2 <= n <= 1000

方法一：

[分析](https://leetcode-cn.com/problems/jian-sheng-zi-ii-lcof/solution/mian-shi-ti-14-ii-jian-sheng-zi-iitan-xin-er-fen-f/)

```java
class Solution {
    public int cuttingRope(int n) {
        if(n <= 3) return n - 1;
        int b = n % 3, p = 1000000007;
        long rem = 1, x = 3;
        for(int a = n / 3 - 1; a > 0; a /= 2) {
            if(a % 2 == 1) rem = (rem * x) % p;
            x = (x * x) % p;
        }
        if(b == 0) return (int)(rem * 3 % p);
        if(b == 1) return (int)(rem * 4 % p);
        return (int)(rem * 6 % p);
    }
}
```

- 时间复杂度 O(logN) ： 其中 N=a ，二分法为对数级别复杂度，每轮仅有求整、求余、次方运算。
  - 求整和求余运算：资料提到不超过机器数的整数可以看作是 O(1)；
  - 幂运算：查阅资料，提到浮点取幂为 O(1) 。
- 空间复杂度 O(1)： 变量 a, b, p, x, rem 使用常数大小额外空间。

### [030] 把字符串转换成整数※

写一个函数 StrToInt，实现把字符串转换成整数这个功能。不能使用 atoi 或者其他类似的库函数。

首先，该函数会根据需要丢弃无用的开头空格字符，直到寻找到第一个非空格的字符为止。

当我们寻找到的第一个非空字符为正或者负号时，则将该符号与之后面尽可能多的连续数字组合起来，作为该整数的正负号；假如第一个非空字符是数字，则直接将其与之后连续的数字字符组合起来，形成整数。

该字符串除了有效的整数部分之后也可能会存在多余的字符，这些字符可以被忽略，它们对于函数不应该造成影响。

注意：假如该字符串中的第一个非空格字符不是一个有效整数字符、字符串为空或字符串仅包含空白字符时，则你的函数不需要进行转换。

在任何情况下，若函数不能进行有效的转换时，请返回 0。

说明：

假设我们的环境只能存储 32 位大小的有符号整数，那么其数值范围为 [−231,  231 − 1]。如果数值超过这个范围，请返回  INT_MAX (231 − 1) 或 INT_MIN (−231) 。

示例 1:

```java
输入: "42"
输出: 42
```

示例 2:

```
输入: "   -42"
输出: -42
解释: 第一个非空白字符为 '-', 它是一个负号。
     我们尽可能将负号与后面所有连续出现的数字组合起来，最后得到 -42 。
```

示例 3:

```
输入: "4193 with words"
输出: 4193
解释: 转换截止于数字 '3' ，因为它的下一个字符不为数字。
```

示例 4:

```
输入: "words and 987"
输出: 0
解释: 第一个非空字符是 'w', 但它不是数字或正、负号。
     因此无法执行有效的转换。
```

示例 5:

```
输入: "-91283472332"
输出: -2147483648
解释: 数字 "-91283472332" 超过 32 位有符号整数范围。 
     因此返回 INT_MIN (−2^31) 。
```

方法一：

[分析](https://leetcode-cn.com/problems/ba-zi-fu-chuan-zhuan-huan-cheng-zheng-shu-lcof/solution/mian-shi-ti-67-ba-zi-fu-chuan-zhuan-huan-cheng-z-4/)

```java
class Solution {
    public int strToInt(String str) {
        int res = 0, bndry = Integer.MAX_VALUE / 10;
        int i = 0, sign = 1, length = str.length();
        if(length == 0) return 0;
        while(str.charAt(i) == ' ')
            if(++i == length) return 0;
        if(str.charAt(i) == '-') sign = -1;
        if(str.charAt(i) == '-' || str.charAt(i) == '+') i++;
        for(int j = i; j < length; j++) {
            if(str.charAt(j) < '0' || str.charAt(j) > '9') break;
            if(res > bndry || res == bndry && str.charAt(j) > '7')
                return sign == 1 ? Integer.MAX_VALUE : Integer.MIN_VALUE;
            res = res * 10 + (str.charAt(j) - '0');
        }
        return sign * res;
    }
}
```

- 时间复杂度 O(N) ： 其中 N 为字符串长度，线性遍历字符串占用 O(N) 时间。
- 空间复杂度 O(N) ： 删除首尾空格后需建立新字符串，最差情况下占用 O(N) 额外空间。

### [031] 表示数值的字符串※

请实现一个函数用来判断字符串是否表示数值（包括整数和小数）。例如，字符串"+100"、"5e2"、"-123"、"3.1416"、"-1E-16"、"0123"都表示数值，但"12e"、"1a3.14"、"1.2.3"、"+-5"及"12e+5.4"都不是。

方法一：

[分析](https://leetcode-cn.com/problems/biao-shi-shu-zhi-de-zi-fu-chuan-lcof/solution/biao-shi-shu-zhi-de-zi-fu-chuan-by-leetcode-soluti/)

```java
class Solution {
    public boolean isNumber(String s) {
        Map<State, Map<CharType, State>> transfer = new HashMap<State, Map<CharType, State>>();
        Map<CharType, State> initialMap = new HashMap<CharType, State>() {{
            put(CharType.CHAR_SPACE, State.STATE_INITIAL);
            put(CharType.CHAR_NUMBER, State.STATE_INTEGER);
            put(CharType.CHAR_POINT, State.STATE_POINT_WITHOUT_INT);
            put(CharType.CHAR_SIGN, State.STATE_INT_SIGN);
        }};
        transfer.put(State.STATE_INITIAL, initialMap);
        Map<CharType, State> intSignMap = new HashMap<CharType, State>() {{
            put(CharType.CHAR_NUMBER, State.STATE_INTEGER);
            put(CharType.CHAR_POINT, State.STATE_POINT_WITHOUT_INT);
        }};
        transfer.put(State.STATE_INT_SIGN, intSignMap);
        Map<CharType, State> integerMap = new HashMap<CharType, State>() {{
            put(CharType.CHAR_NUMBER, State.STATE_INTEGER);
            put(CharType.CHAR_EXP, State.STATE_EXP);
            put(CharType.CHAR_POINT, State.STATE_POINT);
            put(CharType.CHAR_SPACE, State.STATE_END);
        }};
        transfer.put(State.STATE_INTEGER, integerMap);
        Map<CharType, State> pointMap = new HashMap<CharType, State>() {{
            put(CharType.CHAR_NUMBER, State.STATE_FRACTION);
            put(CharType.CHAR_EXP, State.STATE_EXP);
            put(CharType.CHAR_SPACE, State.STATE_END);
        }};
        transfer.put(State.STATE_POINT, pointMap);
        Map<CharType, State> pointWithoutIntMap = new HashMap<CharType, State>() {{
            put(CharType.CHAR_NUMBER, State.STATE_FRACTION);
        }};
        transfer.put(State.STATE_POINT_WITHOUT_INT, pointWithoutIntMap);
        Map<CharType, State> fractionMap = new HashMap<CharType, State>() {{
            put(CharType.CHAR_NUMBER, State.STATE_FRACTION);
            put(CharType.CHAR_EXP, State.STATE_EXP);
            put(CharType.CHAR_SPACE, State.STATE_END);
        }};
        transfer.put(State.STATE_FRACTION, fractionMap);
        Map<CharType, State> expMap = new HashMap<CharType, State>() {{
            put(CharType.CHAR_NUMBER, State.STATE_EXP_NUMBER);
            put(CharType.CHAR_SIGN, State.STATE_EXP_SIGN);
        }};
        transfer.put(State.STATE_EXP, expMap);
        Map<CharType, State> expSignMap = new HashMap<CharType, State>() {{
            put(CharType.CHAR_NUMBER, State.STATE_EXP_NUMBER);
        }};
        transfer.put(State.STATE_EXP_SIGN, expSignMap);
        Map<CharType, State> expNumberMap = new HashMap<CharType, State>() {{
            put(CharType.CHAR_NUMBER, State.STATE_EXP_NUMBER);
            put(CharType.CHAR_SPACE, State.STATE_END);
        }};
        transfer.put(State.STATE_EXP_NUMBER, expNumberMap);
        Map<CharType, State> endMap = new HashMap<CharType, State>() {{
            put(CharType.CHAR_SPACE, State.STATE_END);
        }};
        transfer.put(State.STATE_END, endMap);

        int length = s.length();
        State state = State.STATE_INITIAL;

        for (int i = 0; i < length; i++) {
            CharType type = toCharType(s.charAt(i));
            if (!transfer.get(state).containsKey(type)) {
                return false;
            } else {
                state = transfer.get(state).get(type);
            }
        }
        return state == State.STATE_INTEGER || state == State.STATE_POINT || state == State.STATE_FRACTION || state == State.STATE_EXP_NUMBER || state == State.STATE_END;
    }

    public CharType toCharType(char ch) {
        if (ch >= '0' && ch <= '9') {
            return CharType.CHAR_NUMBER;
        } else if (ch == 'e' || ch == 'E') {
            return CharType.CHAR_EXP;
        } else if (ch == '.') {
            return CharType.CHAR_POINT;
        } else if (ch == '+' || ch == '-') {
            return CharType.CHAR_SIGN;
        } else if (ch == ' ') {
            return CharType.CHAR_SPACE;
        } else {
            return CharType.CHAR_ILLEGAL;
        }
    }

    enum State {
        STATE_INITIAL,
        STATE_INT_SIGN,
        STATE_INTEGER,
        STATE_POINT,
        STATE_POINT_WITHOUT_INT,
        STATE_FRACTION,
        STATE_EXP,
        STATE_EXP_SIGN,
        STATE_EXP_NUMBER,
        STATE_END,
    }

    enum CharType {
        CHAR_NUMBER,
        CHAR_EXP,
        CHAR_POINT,
        CHAR_SIGN,
        CHAR_SPACE,
        CHAR_ILLEGAL,
    }
}
```

时间复杂度：O(N)，其中 N 为字符串的长度。我们需要遍历字符串的每个字符，其中状态转移所需的时间复杂度为 O(1)。
空间复杂度：O(1)。只需要创建固定大小的状态转移表。