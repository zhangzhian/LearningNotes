# Leetcode热题100

### [001] 子集

给定一组不含重复元素的整数数组 nums，返回该数组所有可能的子集（幂集）。

说明：解集不能包含重复的子集。

示例:

```
输入: nums = [1,2,3]
输出:
[
  [3],
  [1],
  [2],
  [1,2,3],
  [1,3],
  [2,3],
  [1,2],
  []
]
```

[题解](https://leetcode-cn.com/problems/subsets)

方法一：迭代法实现子集枚举

```java
class Solution {
    List<Integer> t = new ArrayList<Integer>();
    List<List<Integer>> ans = new ArrayList<List<Integer>>();

    public List<List<Integer>> subsets(int[] nums) {
        int n = nums.length;
        for (int mask = 0; mask < (1 << n); ++mask) {
            t.clear();
            for (int i = 0; i < n; ++i) {
                if ((mask & (1 << i)) != 0) {
                    t.add(nums[i]);
                }
            }
            ans.add(new ArrayList<Integer>(t));
        }
        return ans;
    }
}
```

时间复杂度：
$$
O(n \times 2^n)
$$
空间复杂度：O(n)。即构造子集使用的临时数组 t 的空间代价。

方法二：递归法实现子集枚举

```java
class Solution {
    List<Integer> t = new ArrayList<Integer>();
    List<List<Integer>> ans = new ArrayList<List<Integer>>();

    public List<List<Integer>> subsets(int[] nums) {
        dfs(0, nums);
        return ans;
    }

    public void dfs(int cur, int[] nums) {
        if (cur == nums.length) {
            ans.add(new ArrayList<Integer>(t));
            return;
        }
        t.add(nums[cur]);
        dfs(cur + 1, nums);
        t.remove(t.size() - 1);
        dfs(cur + 1, nums);
    }
}
```

时间复杂度：
$$
O(n \times 2^n)
$$
空间复杂度：O(n)。即构造子集使用的临时数组 t 的空间代价。

方法三：非递归

```java
public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> res = new ArrayList<>(1 << nums.length);
    //先添加一个空的集合
    res.add(new ArrayList<>());
    for (int num : nums) {
        //每遍历一个元素就在之前子集中的每个集合追加这个元素，让他变成新的子集
        for (int i = 0, j = res.size(); i < j; i++) {
            //遍历之前的子集，重新封装成一个新的子集
            List<Integer> list = new ArrayList<>(res.get(i));
            //然后在新的子集后面追加这个元素
            list.add(num);
            //把这个新的子集添加到集合中
            res.add(list);
        }
    }
    return res;
}
```

### [002] 全排列 

给定一个 没有重复 数字的序列，返回其所有可能的全排列。

示例:

```
输入: [1,2,3]
输出:
[
  [1,2,3],
  [1,3,2],
  [2,1,3],
  [2,3,1],
  [3,1,2],
  [3,2,1]
]
```

[题解](https://leetcode-cn.com/problems/permutations/solution/quan-pai-lie-by-leetcode-solution-2/)

方法一：

```java
class Solution {
    public List<List<Integer>> permute(int[] nums) {
        List<List<Integer>> res = new ArrayList<List<Integer>>();

        List<Integer> output = new ArrayList<Integer>();
        for (int num : nums) {
            output.add(num);
        }

        int n = nums.length;
        backtrack(n, output, res, 0);
        return res;
    }

    public void backtrack(int n, List<Integer> output, List<List<Integer>> res, int first) {
        // 所有数都填完了
        if (first == n) {
            res.add(new ArrayList<Integer>(output));
        }
        for (int i = first; i < n; i++) {
            // 动态维护数组
            Collections.swap(output, first, i);
            // 继续递归填下一个数
            backtrack(n, output, res, first + 1);
            // 撤销操作
            Collections.swap(output, first, i);
        }
    }
}
```

时间复杂度：O(n * n!)

空间复杂度：O(n)

### [003] 括号生成

数字 n 代表生成括号的对数，请你设计一个函数，用于能够生成所有可能的并且 有效的 括号组合。

示例：

```java
输入：n = 3
输出：[
       "((()))",
       "(()())",
       "(())()",
       "()(())",
       "()()()"
     ]
```

[题解](https://leetcode-cn.com/problems/generate-parentheses/solution/)

方法一：暴力法

生成所有 2^2n 个 `'('` 和 `')'` 字符构成的序列，然后我们检查每一个是否有效即可。



```java
class Solution {
    public List<String> generateParenthesis(int n) {
        List<String> combinations = new ArrayList<String>();
        generateAll(new char[2 * n], 0, combinations);
        return combinations;
    }

    public void generateAll(char[] current, int pos, List<String> result) {
        if (pos == current.length) {
            if (valid(current)) {
                result.add(new String(current));
            }
        } else {
            current[pos] = '(';
            generateAll(current, pos + 1, result);
            current[pos] = ')';
            generateAll(current, pos + 1, result);
        }
    }

    public boolean valid(char[] current) {
        int balance = 0;
        for (char c: current) {
            if (c == '(') {
                ++balance;
            } else {
                --balance;
            }
            if (balance < 0) {
                return false;
            }
        }
        return balance == 0;
    }
}
```

- 时间复杂度：O(2^2n * n)

- 空间复杂度：O(n)

方法二：回溯法

方法一还有改进的余地：我们可以只在序列仍然保持有效时才添加 '(' or ')'，而不是像 方法一 那样每次添加。我们可以通过跟踪到目前为止放置的左括号和右括号的数目来做到这一点，

如果左括号数量不大于 n，我们可以放一个左括号。如果右括号数量小于左括号的数量，我们可以放一个右括号。



```java
class Solution {
    public List<String> generateParenthesis(int n) {
        List<String> ans = new ArrayList<String>();
        backtrack(ans, new StringBuilder(), 0, 0, n);
        return ans;
    }

    public void backtrack(List<String> ans, StringBuilder cur, int open, int close, int max) {
        if (cur.length() == max * 2) {
            ans.add(cur.toString());
            return;
        }
        if (open < max) {
            cur.append('(');
            backtrack(ans, cur, open + 1, close, max);
            cur.deleteCharAt(cur.length() - 1);
        }
        if (close < open) {
            cur.append(')');
            backtrack(ans, cur, open, close + 1, max);
            cur.deleteCharAt(cur.length() - 1);
        }
    }
}
```

时间复杂度：
$$
O(\dfrac{4^n}{\sqrt{n}})
$$
空间复杂度：O(n) 

方法三：按括号序列的长度递归

任何一个括号序列都一定是由 ( 开头，并且第一个 ( 一定有一个唯一与之对应的 )。这样一来，每一个括号序列可以用 (a)b 来表示，其中 a 与 b 分别是一个合法的括号序列（可以为空）。

那么，要生成所有长度为 2 * n 的括号序列，我们定义一个函数 generate(n) 来返回所有可能的括号序列。那么在函数 generate(n) 的过程中：

- 我们需要枚举与第一个 ( 对应的 ) 的位置 2 * i + 1；
- 递归调用 generate(i) 即可计算 a 的所有可能性；
- 递归调用 generate(n - i - 1) 即可计算 b 的所有可能性；
- 遍历 a 与 b 的所有可能性并拼接，即可得到所有长度为 2 * n 的括号序列。

为了节省计算时间，我们在每次 generate(i) 函数返回之前，把返回值存储起来，下次再调用 generate(i) 时可以直接返回，不需要再递归计算。

```java
class Solution {
    ArrayList[] cache = new ArrayList[100];

    public List<String> generate(int n) {
        if (cache[n] != null) {
            return cache[n];
        }
        ArrayList<String> ans = new ArrayList<String>();
        if (n == 0) {
            ans.add("");
        } else {
            for (int c = 0; c < n; ++c) {
                for (String left: generate(c)) {
                    for (String right: generate(n - 1 - c)) {
                        ans.add("(" + left + ")" + right);
                    }
                }
            }
        }
        cache[n] = ans;
        return ans;
    }

    public List<String> generateParenthesis(int n) {
        return generate(n);
    }
}
```

时间复杂度：
$$
O(\dfrac{4^n}{\sqrt{n}})
$$
空间复杂度：
$$
O(\dfrac{4^n}{\sqrt{n}})
$$

### [004] 比特位计数

给定一个非负整数 num。对于 0 ≤ i ≤ num 范围中的每个数字 i ，计算其二进制数中的 1 的数目并将它们作为数组返回。

示例 1:

```
输入: 2
输出: [0,1,1]
```

示例 2:

```
输入: 5
输出: [0,1,1,2,1,2]
```

进阶:

- 给出时间复杂度为O(n*sizeof(integer))的解答非常容易。但你可以在线性时间O(n)内用一趟扫描做到吗？
- 要求算法的空间复杂度为O(n)。
- 你能进一步完善解法吗？要求在C++或任何其他语言中不使用任何内置函数（如 C++ 中的 __builtin_popcount）来执行此操作。

[题解](https://leetcode-cn.com/problems/counting-bits/solution/)

**方法一：Pop count**

看做 位1的个数 的后续。它计数一个无符号整数的位。结果称为 pop count，或 汉明权重。

```java
public class Solution {
    public int[] countBits(int num) {
        int[] ans = new int[num + 1];
        for (int i = 0; i <= num; ++i)
            ans[i] = popcount(i);
        return ans;
    }
    private int popcount(int x) {
        int count;
        for (count = 0; x != 0; ++count)
          x &= x - 1; //zeroing out the least significant nonzero bit
        return count;
    }
}
```

复杂度分析

- 时间复杂度：O(nk)。对于每个整数 x，我们需要 O(k) 次操作，其中 k 是 x 的位数。
- 空间复杂度：O(n)。 我们需要 O(n)的空间来存储计数结果。如果排除这一点，就只需要常数空间。

**方法二：动态规划 + 最高有效位** 

以二进制形式检查 \[0, 3\]的范围：

$$
(0) = (0)_2
$$

$$
(1) = (1)_2
$$

$$
(2) = (10)_2
$$

$$
(3) = (11)_2
$$

可以看出， 2 和 3 的二进制形式可以通过给 0 和 1 的二进制形式在前面加上 1 来得到。因此，它们的 pop count 只相差 1。

类似的，我们可以使用 [0, 3]作为蓝本来得到 [4, 7]

总之，对于pop count P(x)P(x)，我们有以下的状态转移函数：

$$
P(x + b) = P(x) + 1, b = 2^m > x
$$

```java
public class Solution {
    public int[] countBits(int num) {
        int[] ans = new int[num + 1];
        int i = 0, b = 1;
        // [0, b) is calculated
        while (b <= num) {
            // generate [b, 2b) or [b, num) from [0, b)
            while(i < b && i + b <= num){
                ans[i + b] = ans[i] + 1;
                ++i;
            }
            i = 0;   // reset i
            b <<= 1; // b = 2b
        }
        return ans;
    }
}
```

- 时间复杂度：O(n) 
- 空间复杂度：O(n) 

**方法三：动态规划 + 最低有效位**

遵循上一方法的相同原则，我们还可以通过最低有效位来获得状态转移函数。

观察x 和 x' = x / 2 的关系：
$$
x=(1001011101)_2 =(605)_{10}
$$

$$
x' = (100101110)_2 = (302)_{10}
$$

可以发现 x' 与 x 只有一位不同，这是因为x' 可以看做 x 移除最低有效位的结果。

这样，我们就有了下面的状态转移函数：
$$
P(x)=P(x/2)+(xmod2)
$$

```java
public class Solution {
  public int[] countBits(int num) {
      int[] ans = new int[num + 1];
      for (int i = 1; i <= num; ++i)
        ans[i] = ans[i >> 1] + (i & 1); // x / 2 is x >> 1 and x % 2 is x & 1
      return ans;
  }
}
```

时间复杂度：O(n) 。对每个整数 x ，我们只需要常数时间。
空间复杂度：O(n) 。与方法二相同。

**方法四：动态规划 + 最后设置位**

与上述方法思路相同，我们可以利用最后设置位。

最后设置位是从右到左第一个为1的位。使用 x &= x - 1 将该位设置为0，就可以得到以下状态转移函数：

$$
P(x) = P(x \mathrel{\&} (x - 1)) + 1;
$$

```java
public class Solution {
  public int[] countBits(int num) {
      int[] ans = new int[num + 1];
      for (int i = 1; i <= num; ++i)
        ans[i] = ans[i & (i - 1)] + 1;
      return ans;
  }
}
```

时间复杂度：O(n) 。对每个整数 x ，我们只需要常数时间。
空间复杂度：O(n) 。与方法二相同。

### [005] 二叉树的中序遍历

给定一个二叉树的根节点 root ，返回它的 中序 遍历。

 

示例 1：

![img](https://assets.leetcode.com/uploads/2020/09/15/inorder_1.jpg)

```
输入：root = [1,null,2,3]
输出：[1,3,2]
```

示例 2：

```
输入：root = []
输出：[]
```

示例 3：

```
输入：root = [1]
输出：[1]
```

示例 4：

![img](https://assets.leetcode.com/uploads/2020/09/15/inorder_5.jpg)

```
输入：root = [1,2]
输出：[2,1]
```

示例 5：

![img](https://assets.leetcode.com/uploads/2020/09/15/inorder_4.jpg)

```
输入：root = [1,null,2]
输出：[1,2]
```


提示：

- 树中节点数目在范围 [0, 100] 内
- -100 <= Node.val <= 100


进阶: 递归算法很简单，你可以通过迭代算法完成吗？

方法一：递归

```java
class Solution {
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<Integer>();
        inorder(root, res);
        return res;
    }

    public void inorder(TreeNode root, List<Integer> res) {
        if (root == null) {
            return;
        }
        inorder(root.left, res);
        res.add(root.val);
        inorder(root.right, res);
    }
}
```

- 时间复杂度：O(n) 

- 空间复杂度：O(n) 

方法二：栈（迭代）

```java
class Solution {
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<Integer>();
        Deque<TreeNode> stk = new LinkedList<TreeNode>();
        while (root != null || !stk.isEmpty()) {
            while (root != null) {
                stk.push(root);
                root = root.left;
            }
            root = stk.pop();
            res.add(root.val);
            root = root.right;
        }
        return res;
    }
}
```

- 时间复杂度：O(n) 

- 空间复杂度：O(n) 

方法三：Morris 中序遍历

**Morris 遍历算法**是另一种遍历二叉树的方法，它能将非递归的中序遍历空间复杂度降为 O(1) 。

Morris 遍历算法整体步骤如下（假设当前遍历到的节点为 x）：

1. 如果 x 无左孩子，先将 x 的值加入答案数组，再访问 x 的右孩子，即 x*=*x*.*right。

2. 如果 x 有左孩子，则找到 x 左子树上最右的节点（即左子树中序遍历的最后一个节点，x 在中序遍历中的前驱节点），我们记为 predecessor。根据predecessor 的右孩子是否为空，进行如下操作。
   - 如果 predecessor 的右孩子为空，则将其右孩子指向 x，然后访问 x 的左孩子，即 x = x.left。
   - 如果predecessor 的右孩子不为空，则此时其右孩子指向 x，说明我们已经遍历完 x 的左子树，我们将predecessor 的右孩子置空，将 x 的值加入答案数组，然后访问 x 的右孩子，即 x = x.right。

3. 重复上述操作，直至访问完整棵树。

```java
class Solution {
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<Integer>();
        TreeNode predecessor = null;

        while (root != null) {
            if (root.left != null) {
                // predecessor 节点就是当前 root 节点向左走一步，然后一直向右走至无法走为止
                predecessor = root.left;
                while (predecessor.right != null && predecessor.right != root) {
                    predecessor = predecessor.right;
                }
                
                // 让 predecessor 的右指针指向 root，继续遍历左子树
                if (predecessor.right == null) {
                    predecessor.right = root;
                    root = root.left;
                }
                // 说明左子树已经访问完了，我们需要断开链接
                else {
                    res.add(root.val);
                    predecessor.right = null;
                    root = root.right;
                }
            }
            // 如果没有左孩子，则直接访问右孩子
            else {
                res.add(root.val);
                root = root.right;
            }
        }
        return res;
    }
}
```

- 时间复杂度：O(n) 

- 空间复杂度：O(1) 

### [006] 组合总和

给定一个无重复元素的数组 candidates 和一个目标数 target ，找出 candidates 中所有可以使数字和为 target 的组合。

candidates 中的数字可以无限制重复被选取。

说明：

- 所有数字（包括 target）都是正整数。
- 解集不能包含重复的组合。 

示例 1：

```
输入：candidates = [2,3,6,7], target = 7,
所求解集为：
[
  [7],
  [2,2,3]
]
```

示例 2：

```
输入：candidates = [2,3,5], target = 8,
所求解集为：
[
  [2,2,2,2],
  [2,3,3],
  [3,5]
]
```


提示：

- 1 <= candidates.length <= 30
- 1 <= candidates[i] <= 200
- candidate 中的每个元素都是独一无二的。
- 1 <= target <= 500

[题解](https://leetcode-cn.com/problems/combination-sum/solution/hui-su-suan-fa-jian-zhi-python-dai-ma-java-dai-m-2/)

思路分析：根据示例 1：输入: candidates = [2, 3, 6, 7]，target = 7。

候选数组里有 2，如果找到了组合总和为 7 - 2 = 5 的所有组合，再在之前加上 2 ，就是 7 的所有组合；
同理考虑 3，如果找到了组合总和为 7 - 3 = 4 的所有组合，再在之前加上 3 ，就是 7 的所有组合，依次这样找下去。

这一类问题都需要先画出树形图，然后编码实现。

编码通过 深度优先遍历 实现，使用一个列表，在 深度优先遍历 变化的过程中，遍历所有可能的列表并判断当前列表是否符合题目的要求，成为「回溯算法」

![img](https://pic.leetcode-cn.com/1598091943-hZjibJ-file_1598091940241)

这棵树有 44 个叶子结点的值 0，对应的路径列表是 [[2, 2, 3], [2, 3, 2], [3, 2, 2], [7]]，而示例中给出的输出只有 [[7], [2, 2, 3]]。即：题目中要求每一个符合要求的解是 不计算顺序 的。下面我们分析为什么会产生重复。

产生重复的原因是：在每一个结点，做减法，展开分支的时候，由于题目中说 **每一个元素可以重复使用**，我们考虑了 **所有的** 候选数，因此出现了重复的列表。

遇到这一类相同元素不计算顺序的问题，我们在搜索的时候就需要 **按某种顺序搜索**。具体的做法是：每一次搜索的时候设置 **下一轮搜索的起点** `begin`，请看下图。

![img](https://pic.leetcode-cn.com/1598091943-GPoHAJ-file_1598091940246)

即：从每一层的第 2 个结点开始，都不能再搜索产生同一层结点已经使用过的 `candidate` 里的元素。



方法一：回溯

```java
import java.util.ArrayDeque;
import java.util.ArrayList;
import java.util.Deque;
import java.util.List;

public class Solution {

    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        int len = candidates.length;
        List<List<Integer>> res = new ArrayList<>();
        if (len == 0) {
            return res;
        }

        Deque<Integer> path = new ArrayDeque<>();
        dfs(candidates, 0, len, target, path, res);
        return res;
    }

    /**
     * @param candidates 候选数组
     * @param begin      搜索起点
     * @param len        冗余变量，是 candidates 里的属性，可以不传
     * @param target     每减去一个元素，目标值变小
     * @param path       从根结点到叶子结点的路径，是一个栈
     * @param res        结果集列表
     */
    private void dfs(int[] candidates, int begin, int len, int target, Deque<Integer> path, List<List<Integer>> res) {
        // target 为负数和 0 的时候不再产生新的孩子结点
        if (target < 0) {
            return;
        }
        if (target == 0) {
            res.add(new ArrayList<>(path));
            return;
        }

        // 重点理解这里从 begin 开始搜索的语意
        for (int i = begin; i < len; i++) {
            path.addLast(candidates[i]);

            // 注意：由于每一个元素可以重复使用，下一轮搜索的起点依然是 i，这里非常容易弄错
            dfs(candidates, i, len, target - candidates[i], path, res);

            // 状态重置
            path.removeLast();
        }
    }
}
```

- 时间复杂度：O(S)，其中 S 为所有可行解的长度之和

- 空间复杂度：O(target)。除答案数组外，空间复杂度取决于递归的栈深度，在最差情况下需要递归 O(target) 层。

方法二：回溯+枝减

根据上面画树形图的经验，如果 target 减去一个数得到负数，那么减去一个更大的树依然是负数，同样搜索不到结果。基于这个想法，我们可以对输入数组进行排序，添加相关逻辑达到进一步剪枝的目的；

排序是为了提高搜索速度，对于解决这个问题来说非必要。但是搜索问题一般复杂度较高，能剪枝就尽量剪枝。实际工作中如果遇到两种方案拿捏不准的情况，都试一下。

```java
import java.util.ArrayDeque;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Deque;
import java.util.List;

public class Solution {

    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        int len = candidates.length;
        List<List<Integer>> res = new ArrayList<>();
        if (len == 0) {
            return res;
        }

        // 排序是剪枝的前提
        Arrays.sort(candidates);
        Deque<Integer> path = new ArrayDeque<>();
        dfs(candidates, 0, len, target, path, res);
        return res;
    }

    private void dfs(int[] candidates, int begin, int len, int target, Deque<Integer> path, List<List<Integer>> res) {
        // 由于进入更深层的时候，小于 0 的部分被剪枝，因此递归终止条件值只判断等于 0 的情况
        if (target == 0) {
            res.add(new ArrayList<>(path));
            return;
        }

        for (int i = begin; i < len; i++) {
            // 重点理解这里剪枝，前提是候选数组已经有序，
            if (target - candidates[i] < 0) {
                break;
            }
            
            path.addLast(candidates[i]);
            dfs(candidates, i, len, target - candidates[i], path, res);
            path.removeLast();
        }
    }
}
```

### [007] 二叉树展开为链表

给定一个二叉树，原地将它展开为一个单链表。

 例如，给定二叉树

    	1
       / \
      2   5
     / \   \
    3   4   6

将其展开为：

```
1
 \
  2
   \
    3
     \
      4
       \
        5
         \
          6
```

方法一：DFS 递归
![114_1.png](https://pic.leetcode-cn.com/7427f6e30a8a6e3d44375579d00b9e6eec53500b67643868817b7ad775b82adb-114_1.png)

其实是分为三步：

- 首先将根节点的左子树变成链表
- 其次将根节点的右子树变成链表
- 最后将变成链表的右子树放在变成链表的左子树的最右边

```java
class Solution {
    public void flatten(TreeNode root) {
        if(root == null){
            return ;
        }
        //将根节点的左子树变成链表
        flatten(root.left);
        //将根节点的右子树变成链表
        flatten(root.right);
        TreeNode temp = root.right;
        //把树的右边换成左边的链表
        root.right = root.left;
        //记得要将左边置空
        root.left = null;
        //找到树的最右边的节点
        while(root.right != null) root = root.right;
        //把右边的链表接到刚才树的最右边的节点
        root.right = temp;
    }
}
```

- 时间复杂度：O(n) 

- 空间复杂度：O(n) 

方法二：前序遍历

可以对二叉树进行前序遍历，获得各节点被访问到的顺序。由于将二叉树展开为链表之后会破坏二叉树的结构，因此在前序遍历结束之后更新每个节点的左右子节点的信息，将二叉树展开为单链表。

递归：

```java
class Solution {
    public void flatten(TreeNode root) {
        List<TreeNode> list = new ArrayList<TreeNode>();
        preorderTraversal(root, list);
        int size = list.size();
        for (int i = 1; i < size; i++) {
            TreeNode prev = list.get(i - 1);
            TreeNode curr = list.get(i);
            prev.left = null;
            prev.right = curr;
        }
    }

    public void preorderTraversal(TreeNode root, List<TreeNode> list) {
        if (root != null) {
            list.add(root);
            preorderTraversal(root.left, list);
            preorderTraversal(root.right, list);
        }
    }
}
```

迭代

```java
class Solution {
    public void flatten(TreeNode root) {
        List<TreeNode> list = new ArrayList<TreeNode>();
        Deque<TreeNode> stack = new LinkedList<TreeNode>();
        TreeNode node = root;
        while (node != null || !stack.isEmpty()) {
            while (node != null) {
                list.add(node);
                stack.push(node);
                node = node.left;
            }
            node = stack.pop();
            node = node.right;
        }
        int size = list.size();
        for (int i = 1; i < size; i++) {
            TreeNode prev = list.get(i - 1), curr = list.get(i);
            prev.left = null;
            prev.right = curr;
        }
    }
}
```

- 时间复杂度：O(n) 

- 空间复杂度：O(n) 

方法三：前序遍历和展开同步进行

每次从栈内弹出一个节点作为当前访问的节点，获得该节点的子节点，如果子节点不为空，则依次将右子节点和左子节点压入栈内（注意入栈顺序）。

展开为单链表的做法是，维护上一个访问的节点 prev，每次访问一个节点时，令当前访问的节点为 curr，将 prev 的左子节点设为 null 以及将 prev 的右子节点设为 curr，然后将 curr 赋值给 prev，进入下一个节点的访问，直到遍历结束。需要注意的是，初始时 prev 为 null，只有在 prev 不为 null 时才能对 prev 的左右子节点进行更新。

```java
class Solution {
    public void flatten(TreeNode root) {
        if (root == null) {
            return;
        }
        Deque<TreeNode> stack = new LinkedList<TreeNode>();
        stack.push(root);
        TreeNode prev = null;
        while (!stack.isEmpty()) {
            TreeNode curr = stack.pop();
            if (prev != null) {
                prev.left = null;
                prev.right = curr;
            }
            TreeNode left = curr.left
            TreeNode right = curr.right;
            if (right != null) {
                stack.push(right);
            }
            if (left != null) {
                stack.push(left);
            }
            prev = curr;
        }
    }
}
```

- 时间复杂度：O(n) 

- 空间复杂度：O(n) 

方法四：寻找前驱节点

注意到前序遍历访问各节点的顺序是根节点、左子树、右子树。如果一个节点的左子节点为空，则该节点不需要进行展开操作。如果一个节点的左子节点不为空，则该节点的左子树中的最后一个节点被访问之后，该节点的右子节点被访问。该节点的左子树中最后一个被访问的节点是左子树中的最右边的节点，也是该节点的前驱节点。因此，问题转化成寻找当前节点的前驱节点。

具体做法是，对于当前节点，如果其左子节点不为空，则在其左子树中找到最右边的节点，作为前驱节点，将当前节点的右子节点赋给前驱节点的右子节点，然后将当前节点的左子节点赋给当前节点的右子节点，并将当前节点的左子节点设为空。对当前节点处理结束后，继续处理链表中的下一个节点，直到所有节点都处理结束。

```java
class Solution {
    public void flatten(TreeNode root) {
        TreeNode curr = root;
        while (curr != null) {
            if (curr.left != null) {
                TreeNode next = curr.left;
                TreeNode predecessor = next;
                while (predecessor.right != null) {
                    predecessor = predecessor.right;
                }
                predecessor.right = curr.right;
                curr.left = null;
                curr.right = next;
            }
            curr = curr.right;
        }
    }
}
```

复杂度分析

时间复杂度：O(n)，其中 n 是二叉树的节点数。展开为单链表的过程中，需要对每个节点访问一次，在寻找前驱节点的过程中，每个节点最多被额外访问一次。

空间复杂度：O(1)

### [008] 除自身以外数组的乘积

给你一个长度为 n 的整数数组 nums，其中 n > 1，返回输出数组 output ，其中 output[i] 等于 nums 中除 nums[i] 之外其余各元素的乘积。

示例:

```
输入: [1,2,3,4]
输出: [24,12,8,6]
```


提示：题目数据保证数组之中任意元素的全部前缀元素和后缀（甚至是整个数组）的乘积都在 32 位整数范围内。

说明: 请**不要使用除法**，且在 O(n) 时间复杂度内完成此题。

进阶：你可以在常数空间复杂度内完成这个题目吗？

方法一：左右乘积列表

不必将所有数字的乘积除以给定索引处的数字得到相应的答案，而是利用索引左侧所有数字的乘积和右侧所有数字的乘积（即前缀与后缀）相乘得到答案。

对于给定索引 i，我们将使用它左边所有数字的乘积乘以右边所有数字的乘积。

```java
class Solution {
    public int[] productExceptSelf(int[] nums) {
        int length = nums.length;

        // L 和 R 分别表示左右两侧的乘积列表
        int[] L = new int[length];
        int[] R = new int[length];

        int[] answer = new int[length];

        // L[i] 为索引 i 左侧所有元素的乘积
        // 对于索引为 '0' 的元素，因为左侧没有元素，所以 L[0] = 1
        L[0] = 1;
        for (int i = 1; i < length; i++) {
            L[i] = nums[i - 1] * L[i - 1];
        }

        // R[i] 为索引 i 右侧所有元素的乘积
        // 对于索引为 'length-1' 的元素，因为右侧没有元素，所以 R[length-1] = 1
        R[length - 1] = 1;
        for (int i = length - 2; i >= 0; i--) {
            R[i] = nums[i + 1] * R[i + 1];
        }

        // 对于索引 i，除 nums[i] 之外其余各元素的乘积就是左侧所有元素的乘积乘以右侧所有元素的乘积
        for (int i = 0; i < length; i++) {
            answer[i] = L[i] * R[i];
        }

        return answer;
    }
}
```

- 时间复杂度：O(N) 
- 空间复杂度：O(N) 

方法二：空间复杂度 O(1) 的方法

由于输出数组不算在空间复杂度内，那么我们可以将 L 或 R 数组用输出数组来计算。先把输出数组当作 L 数组来计算，然后再动态构造 R 数组得到结果。让我们来看看基于这个思想的算法。

```java
class Solution {
    public int[] productExceptSelf(int[] nums) {
        int length = nums.length;
        int[] answer = new int[length];

        // answer[i] 表示索引 i 左侧所有元素的乘积
        // 因为索引为 '0' 的元素左侧没有元素， 所以 answer[0] = 1
        answer[0] = 1;
        for (int i = 1; i < length; i++) {
            answer[i] = nums[i - 1] * answer[i - 1];
        }

        // R 为右侧所有元素的乘积
        // 刚开始右边没有元素，所以 R = 1
        int R = 1;
        for (int i = length - 1; i >= 0; i--) {
            // 对于索引 i，左边的乘积为 answer[i]，右边的乘积为 R
            answer[i] = answer[i] * R;
            // R 需要包含右边所有的乘积，所以计算下一个结果时需要将当前值乘到 R 上
            R *= nums[i];
        }
        return answer;
    }
}
```

- 时间复杂度：O(N) 
- 空间复杂度：O(1) 

### [009] 旋转图像

给定一个 n × n 的二维矩阵表示一个图像。

将图像顺时针旋转 90 度。

说明：

你必须在原地旋转图像，这意味着你需要直接修改输入的二维矩阵。请不要使用另一个矩阵来旋转图像。

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

方法 1 ：转置加翻转

先转置矩阵，然后翻转每一行。这个简单的方法已经能达到最优的时间复杂度O(N^2)。

```java
class Solution {
  public void rotate(int[][] matrix) {
    int n = matrix.length;

    // transpose matrix
    for (int i = 0; i < n; i++) {
      for (int j = i; j < n; j++) {
        int tmp = matrix[j][i];
        matrix[j][i] = matrix[i][j];
        matrix[i][j] = tmp;
      }
    }
    // reverse each row
    for (int i = 0; i < n; i++) {
      for (int j = 0; j < n / 2; j++) {
        int tmp = matrix[i][j];
        matrix[i][j] = matrix[i][n - j - 1];
        matrix[i][n - j - 1] = tmp;
      }
    }
  }
}
```

- 时间复杂度：O(N^2) 
- 空间复杂度：O(1) 

方法 2 ：旋转四个矩形

![图片.png](https://pic.leetcode-cn.com/344be9af9ed8f74d23a1703a495aa812d14570b3a15afb990605ff4c8f404042-%E5%9B%BE%E7%89%87.png)

对每个框框，其实都有 4 个顶点：

![图片.png](https://pic.leetcode-cn.com/c834642c7749aa492982a1bf3ddc3a4dbebe518c8f6d13d75ecf8fa3072a925f-%E5%9B%BE%E7%89%87.png)

剩下的就是交换这四个顶点的值：

![图片.png](https://pic.leetcode-cn.com/b622b50a15760f5f369ac16600c9d1f180f087137a41e2144fbcffc69770ebe9-%E5%9B%BE%E7%89%87.png)

交换完毕之后，再继续交换后四个顶点：

![图片.png](https://pic.leetcode-cn.com/ac5d316abc1b607d3b69d80cb86d454ec92d1b02b50a1152cf4be4f1ad2bf7f3-%E5%9B%BE%E7%89%87.png)

```java
class Solution {
  public void rotate(int[][] matrix) {
    int n = matrix.length;
    for (int i = 0; i < n / 2 + n % 2; i++) {
      for (int j = 0; j < n / 2; j++) {
        int[] tmp = new int[4];
        int row = i;
        int col = j;
        for (int k = 0; k < 4; k++) {
          tmp[k] = matrix[row][col];
          int x = row;
          row = col;
          col = n - 1 - x;
        }
        for (int k = 0; k < 4; k++) {
          matrix[row][col] = tmp[(k + 3) % 4];
          int x = row;
          row = col;
          col = n - 1 - x;
        }
      }
    }
  }
}
```

- 间复杂度：O(N^2) 
- 空间复杂度：O(1) 

方法 3：在单次循环中旋转 4 个矩形

该想法和方法 2 相同，但是所有的操作可以在单次循环内完成并且这是更精简的方法。

```java
class Solution {
  public void rotate(int[][] matrix) {
    int n = matrix.length;
    for (int i = 0; i < (n + 1) / 2; i ++) {
      for (int j = 0; j < n / 2; j++) {
        int temp = matrix[n - 1 - j][i];
        matrix[n - 1 - j][i] = matrix[n - 1 - i][n - j - 1];
        matrix[n - 1 - i][n - j - 1] = matrix[j][n - 1 -i];
        matrix[j][n - 1 - i] = matrix[i][j];
        matrix[i][j] = temp;
      }
    }
  }
}
```

时间复杂度：O(N^2)
空间复杂度：O(1)

### [010] 根据身高重建队列

假设有打乱顺序的一群人站成一个队列，数组 people 表示队列中一些人的属性（不一定按顺序）。每个 `people[i] = [hi, ki]`表示第 i 个人的身高为 hi ，前面 正好 有 ki 个身高大于或等于 hi 的人。

请你重新构造并返回输入数组 people 所表示的队列。返回的队列应该格式化为数组 queue ，其中 `queue[j] = [hj, kj]` 是队列中第 j 个人的属性（queue[0] 是排在队列前面的人）。

示例 1：

```
输入：people = [[7,0],[4,4],[7,1],[5,0],[6,1],[5,2]]
输出：[[5,0],[7,0],[5,2],[6,1],[4,4],[7,1]]
解释：
编号为 0 的人身高为 5 ，没有身高更高或者相同的人排在他前面。
编号为 1 的人身高为 7 ，没有身高更高或者相同的人排在他前面。
编号为 2 的人身高为 5 ，有 2 个身高更高或者相同的人排在他前面，即编号为 0 和 1 的人。
编号为 3 的人身高为 6 ，有 1 个身高更高或者相同的人排在他前面，即编号为 1 的人。
编号为 4 的人身高为 4 ，有 4 个身高更高或者相同的人排在他前面，即编号为 0、1、2、3 的人。
编号为 5 的人身高为 7 ，有 1 个身高更高或者相同的人排在他前面，即编号为 1 的人。
因此 [[5,0],[7,0],[5,2],[6,1],[4,4],[7,1]] 是重新构造后的队列。
```

示例 2：

```
输入：people = [[6,0],[5,0],[4,0],[3,2],[2,2],[1,4]]
输出：[[4,0],[5,0],[2,2],[3,2],[1,4],[6,0]]
```


提示：

- 1 <= people.length <= 2000
- 0 <= hi <= 106
- 0 <= ki < people.length
- 题目数据确保队列可以被重建

方法一·: 从高到低考虑

将每个人按照身高从大到小进行排序，处理身高相同的人使用的方法类似，即：按照 h_i为第一关键字降序，k_i为第二关键字升序进行排序。

然后次将每个人放入队列中，可以采用「插空」的方法，依次给每一个人在当前的队列中选择一个插入的位置。

```java
class Solution {
    public int[][] reconstructQueue(int[][] people) {
        //排序前		：[[7,0],[4,4],[7,1],[5,0],[6,1],[5,2]]
        //people排序后	：[[7,0],[7,1],[6,1],[5,0],[5,2],[4,4]]
        Arrays.sort(people, new Comparator<int[]>() {
            public int compare(int[] person1, int[] person2) {
                if (person1[0] != person2[0]) {
                    return person2[0] - person1[0];
                } else {
                    return person1[1] - person2[1];
                }
            }
        });
        
        List<int[]> ans = new ArrayList<int[]>();
        for (int[] person : people) {
            //在指定位置插入元素
            ans.add(person[1], person);
        }
        //结果[[5,0],[7,0],[5,2],[6,1],[4,4],[7,1]]
        return ans.toArray(new int[ans.size()][]);
    }
}
```

- 时间复杂度：O(n^2)，其中 n 是数组 people 的长度。我们需要 O(nlogn) 的时间进行排序，随后需要 O(n^2)的时间遍历每一个人并将他们放入队列中。由于前者在渐近意义下小于后者，因此总时间复杂度为 O(n^2)。

- 空间复杂度：O(logn)。

### [011] 实现 Trie (前缀树)

实现一个 Trie (前缀树)，包含 insert, search, 和 startsWith 这三个操作。

示例:

```
Trie trie = new Trie();

trie.insert("apple");
trie.search("apple");   // 返回 true
trie.search("app");     // 返回 false
trie.startsWith("app"); // 返回 true
trie.insert("app");   
trie.search("app");     // 返回 true
```

说明:

- 你可以假设所有的输入都是由小写字母 a-z 构成的。
- 保证所有输入均为非空字符串。

方法一：

Trie (发音为 "try") 或前缀树是一种树数据结构，用于检索字符串数据集中的键。

Trie 树的结点结构:

Trie 树是一个有根的树，其结点具有以下字段：。

- 最多 R 个指向子结点的链接，其中每个链接对应字母表数据集中的一个字母。本文中假定 R 为 26，小写拉丁字母的数量。

- 布尔字段，以指定节点是对应键的结尾还是只是键前缀。

**向 Trie 树中插入键**

我们通过搜索 Trie 树来插入一个键。我们从根开始搜索它对应于第一个键字符的链接。有两种情况：

- 链接存在。沿着链接移动到树的下一个子层。算法继续搜索下一个键字符。
- 链接不存在。创建一个新的节点，并将它与父节点的链接相连，该链接与当前的键字符相匹配。

重复以上步骤，直到到达键的最后一个字符，然后将当前节点标记为结束节点，算法完成。

- 时间复杂度：O(m)，其中 m 为键长。在算法的每次迭代中，我们要么检查要么创建一个节点，直到到达键尾。只需要 m 次操作。

- 空间复杂度：O(m)。最坏的情况下，新插入的键和 Trie 树中已有的键没有公共前缀。此时需要添加 m 个结点，使用 O(m) 空间。

**在 Trie 树中查找键**

每个键在 trie 中表示为从根到内部节点或叶的路径。我们用第一个键字符从根开始。检查当前节点中与键字符对应的链接。有两种情况：

- 存在链接。我们移动到该链接后面路径中的下一个节点，并继续搜索下一个键字符。
- 不存在链接。若已无键字符，且当前结点标记为 isEnd，则返回 true。否则有两种可能，均返回 false :
  - 还有键字符剩余，但无法跟随 Trie 树的键路径，找不到键。
  - 没有键字符剩余，但当前结点没有标记为 isEnd。也就是说，待查找键只是Trie树中另一个键的前缀。

- 时间复杂度 : O(m)。算法的每一步均搜索下一个键字符。最坏的情况下需要 m 次操作。
- 空间复杂度 : O(1)。

**查找 Trie 树中的键前缀**

该方法与在 Trie 树中搜索键时使用的方法非常相似。我们从根遍历 Trie 树，直到键前缀中没有字符，或者无法用当前的键字符继续 Trie 中的路径。与上面提到的“搜索键”算法唯一的区别是，到达键前缀的末尾时，总是返回 true。我们不需要考虑当前 Trie 节点是否用 “isend” 标记，因为我们搜索的是键的前缀，而不是整个键。

- 时间复杂度 : O(m)。
- 空间复杂度 : O(1)。

```java
class Trie {
    class TrieNode {
        // R links to node children
        private TrieNode[] links;

        private final int R = 26;

        private boolean isEnd;

        public TrieNode() {
            links = new TrieNode[R];
        }

        public boolean containsKey(char ch) {
            return links[ch -'a'] != null;
        }
        public TrieNode get(char ch) {
            return links[ch -'a'];
        }
        public void put(char ch, TrieNode node) {
            links[ch -'a'] = node;
        }
        public void setEnd() {
            isEnd = true;
        }
        public boolean isEnd() {
            return isEnd;
        }
    }

    private TrieNode root;
    /** Initialize your data structure here. */
    public Trie() {
        root = new TrieNode();
    }
    
    /** Inserts a word into the trie. */
    public void insert(String word) {
        TrieNode node = root;
        for (int i = 0; i < word.length(); i++) {
            char currentChar = word.charAt(i);
            if (!node.containsKey(currentChar)) {
                node.put(currentChar, new TrieNode());
            }
            node = node.get(currentChar);
        }
        node.setEnd();
    }

    private TrieNode searchPrefix(String word) {
        TrieNode node = root;
        for (int i = 0; i < word.length(); i++) {
           char curLetter = word.charAt(i);
           if (node.containsKey(curLetter)) {
               node = node.get(curLetter);
           } else {
               return null;
           }
        }
        return node;
    }

    /** Returns if the word is in the trie. */
    public boolean search(String word) {
        TrieNode node = searchPrefix(word);
        return node != null && node.isEnd();
    }
    
    /** Returns if there is any word in the trie that starts with the given prefix. */
    public boolean startsWith(String prefix) {
        TrieNode node = searchPrefix(prefix);
        return node != null;
    }
}

/**
 * Your Trie object will be instantiated and called as such:
 * Trie obj = new Trie();
 * obj.insert(word);
 * boolean param_2 = obj.search(word);
 * boolean param_3 = obj.startsWith(prefix);
 */
```

### [012] 不同的二叉搜索树

给定一个整数 n，求以 1 ... n 为节点组成的二叉搜索树有多少种？

示例:

```
输入: 3
输出: 5
解释:
给定 n = 3, 一共有 5 种不同结构的二叉搜索树:
	1         3     3      2      1
    \       /     /      / \      \
     3     2     1      1   3      2
    /     /       \                 \
   2     1         2                 3
```

方法一：动态规划

题目要求是计算不同二叉搜索树的个数。为此，我们可以定义两个函数：

1. `G(n)`: 长度为 n 的序列能构成的不同二叉搜索树的个数。

2. `F(i, n)`: 以 i 为根、序列长度为 n 的不同二叉搜索树个数`(1≤i≤n)`。

`G(n)` 是我们求解需要的函数。

同的二叉搜索树的总数 `G(n)`，是对遍历所有 `(1≤i≤n)` 的 `F(i, n)` 之和。换言之：
$$
G(n)= \sum_{i=1}^n F(i,n)
$$
对于边界情况，当序列长度为 1（只有根）或为 0（空树）时，只有一种情况，即：G(0)=1,G(1)=1。

给定序列 1⋯n，我们选择数字 i 作为根，则根为 i 的所有二叉搜索树的集合是左子树集合和右子树集合的笛卡尔积，对于笛卡尔积中的每个元素，加上根节点之后形成完整的二叉搜索树，如下图所示：

![fig1](https://assets.leetcode-cn.com/solution-static/96/96_fig1.png)

举例而言，创建以 3 为根、长度为 7 的不同二叉搜索树，整个序列是 `[1, 2, 3, 4, 5, 6, 7]`，我们需要从左子序列 `[1, 2]` 构建左子树，从右子序列`[4, 5, 6, 7]` 构建右子树，然后将它们组合（即笛卡尔积）。

对于这个例子，不同二叉搜索树的个数为 `F(3, 7)`。我们将 `[1,2]` 构建不同左子树的数量表示为` G(2)`, 从 `[4, 5, 6, 7]` 构建不同右子树的数量表示为`G(4)`，注意到 `G(n)` 和序列的内容无关，只和序列的长度有关。于是，`F(3,7)=G(2)⋅G(4)`。 
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
$$
F(i,n)=G(i−1)⋅G(n−i)
$$
将公式以上结合，可以得到 G(n)的递归表达式：
$$
G(n)= \sum_{i=1}^nG(i−1)⋅G(n−i)
$$
至此，我们从小到大计算G函数即可，因为 G(n)的值依赖于 `G(0) G(0)⋯G(n−1)`

```java
class Solution {
    public int numTrees(int n) {
        int[] G = new int[n + 1];
        G[0] = 1;
        G[1] = 1;

        for (int i = 2; i <= n; ++i) {
            for (int j = 1; j <= i; ++j) {
                G[i] += G[j - 1] * G[i - j];
            }
        }
        return G[n];
    }
}
```

间复杂度 : O(n^2)，其中 n 表示二叉搜索树的节点个数。G(n) 函数一共有 n 个值需要求解，每次求解需要 O(n) 的时间复杂度，因此总时间复杂度为 O(n^2)。

空间复杂度 : O(n)。我们需要 O(n) 的空间存储 G 数组。

方法二：数学

方法一中推导出的 G(n)函数的值在数学上被称为卡塔兰数C_n。卡塔兰数更便于计算的定义如下:
$$
C_0=1,C_{n+1}= \frac{2(2n+1)}{n+2}C_n
$$

```java
class Solution {
    public int numTrees(int n) {
        // 提示：我们在这里需要用 long 类型防止计算过程中的溢出
        long C = 1;
        for (int i = 0; i < n; ++i) {
            C = C * 2 * (2 * i + 1) / (i + 2);
        }
        return (int) C;
    }
}
```

- 时间复杂度 : O(n)，其中 n 表示二叉搜索树的节点个数。我们只需要循环遍历一次即可。
- 空间复杂度 : O(1)。我们只需要常数空间存放若干变量。

### [013] 从前序与中序遍历序列构造二叉树

根据一棵树的前序遍历与中序遍历构造二叉树。

注意:
你可以假设树中没有重复的元素。

例如，给出

```
前序遍历 preorder = [3,9,20,15,7]
中序遍历 inorder = [9,3,15,20,7]
```

返回如下的二叉树：

    	3
       / \
      9  20
        /  \
       15   7
方法一：

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

### [014] 最小路径和

给定一个包含非负整数的 m x n 网格 grid ，请找出一条从左上角到右下角的路径，使得路径上的数字总和为最小。

说明：每次只能向下或者向右移动一步。

示例 1：

![img](https://assets.leetcode.com/uploads/2020/11/05/minpath.jpg)

```
输入：grid = [[1,3,1],[1,5,1],[4,2,1]]
输出：7
解释：因为路径 1→3→1→1→1 的总和最小。
```

示例 2：

```
输入：grid = [[1,2,3],[4,5,6]]
输出：12
```


提示：

```
m == grid.length
n == grid[i].length
1 <= m, n <= 200
0 <= grid[i][j] <= 100
```

方法一：动态规划

**状态定义**

设 dp 为大小 m×n 矩阵，其中 `dp[i][j]` 的值代表直到走到 (i,j) 的最小路径和。

**转移方程**

> 题目要求，只能向右或向下走，换句话说，当前单元格 (i,j) 只能从左方单元格 (i-1,j) 或上方单元格 (i,j-1)走到，因此只需要考虑矩阵左边界和上边界。

走到当前单元格 `(i,j)` 的最小路径和 == “从左方单元格 `(i-1,j)` 与 从上方单元格 `(i,j-1)` 走来的 两个最小路径和中较小的 ” ++ 当前单元格值 `grid[i][j]`，具体分为以下 4 种情况：

- 当左边和上边都不是矩阵边界时： `即当i != 0 , j != 0 时，dp[i][j] = min(dp[i - 1][j], dp[i][j - 1]) + grid[i][j] `
- 当只有左边是矩阵边界时：`只能从上面来，即当i = 0, j != 0 时， dp[i][j] = dp[i][j - 1] + grid[i][j]`
- 当只有上边是矩阵边界时： `只能从左面来，即当i != 0, j = 0时， dp[i][j] = dp[i - 1][j] + grid[i][j] `
- 当左边和上边都是矩阵边界时：`即当i = 0, j = 0 时，其实就是起点， dp[i][j] = grid[i][j]`

**初始状态**

dp 初始化即可，不需要修改初始 0 值。

**返回值**

返回 dp 矩阵右下角值，即走到终点的最小路径和。

> 其实我们完全不需要建立 dp 矩阵浪费额外空间，直接遍历 `grid[i][j]`修改即可。这是因为：`grid[i][j] = min(grid[i - 1][j], grid[i][j - 1]) + grid[i][j] `；原 grid 矩阵元素中被覆盖为 dp 元素后（都处于当前遍历点的左上方），不会再被使用到。
>

```java
class Solution {
    public int minPathSum(int[][] grid) {
        for(int i = 0; i < grid.length; i++) {
            for(int j = 0; j < grid[0].length; j++) {
                if(i == 0 && j == 0) continue;
                else if(i == 0)  grid[i][j] = grid[i][j - 1] + grid[i][j];
                else if(j == 0)  grid[i][j] = grid[i - 1][j] + grid[i][j];
                else grid[i][j] = Math.min(grid[i - 1][j], grid[i][j - 1]) + grid[i][j];
            }
        }
        return grid[grid.length - 1][grid[0].length - 1];
    }
}
```

- 时间复杂度 O(M×N) ： 遍历整个 grid 矩阵元素。
- 空间复杂度 OO(1) ： 直接修改原矩阵，不使用额外空间。

### [015] 排序链表

给你链表的头结点 head ，请将其按 升序 排列并返回 排序后的链表 。

进阶：

你可以在 O(n log n) 时间复杂度和常数级空间复杂度下，对链表进行排序吗？


示例 1：

```
输入：head = [4,2,1,3]
输出：[1,2,3,4]
```

示例 2：

```
输入：head = [-1,5,3,4,0]
输出：[-1,0,3,4,5]
```

示例 3：

```
输入：head = []
输出：[]
```


提示：

- 链表中节点的数目在范围 [0, 5 * 10^4] 内
- -10^5 <= Node.val <= 10^5

方法一：归并排序（从底至顶直接合并）

![Picture1.png](https://pic.leetcode-cn.com/c1d5347aa56648afdec22372ee0ed13cf4c25347bd2bb9727b09327ce04360c2-Picture1.png)

```java
public class Solution {
    // 自底向上归并排序
    public ListNode sortList(ListNode head) {
        if(head == null){
            return head;
        }

        // 1. 首先从头向后遍历,统计链表长度
        int length = 0; // 用于统计链表长度
        ListNode node = head;
        while(node != null){
            length++;
            node = node.next;
        }

        // 2. 初始化 引入dummynode
        ListNode dummyHead = new ListNode(0);
        dummyHead.next = head;

        // 3. 每次将链表拆分成若干个长度为subLen的子链表 , 并按照每两个子链表一组进行合并
        for(int subLen = 1;subLen < length;subLen <<= 1){ // subLen每次左移一位（即sublen = sublen*2） PS:位运算对CPU来说效率更高
            ListNode prev = dummyHead;
            ListNode curr = dummyHead.next;     // curr用于记录拆分链表的位置

            while(curr != null){               // 如果链表没有被拆完
                // 3.1 拆分subLen长度的链表1
                ListNode head_1 = curr;        // 第一个链表的头 即 curr初始的位置
                for(int i = 1; i < subLen && curr != null && curr.next != null; i++){     // 拆分出长度为subLen的链表1
                    curr = curr.next;
                }

                // 3.2 拆分subLen长度的链表2
                ListNode head_2 = curr.next;  // 第二个链表的头  即 链表1尾部的下一个位置
                curr.next = null;             // 断开第一个链表和第二个链表的链接
                curr = head_2;                // 第二个链表头 重新赋值给curr
                for(int i = 1;i < subLen && curr != null && curr.next != null;i++){      // 再拆分出长度为subLen的链表2
                    curr = curr.next;
                }

                // 3.3 再次断开 第二个链表最后的next的链接
                ListNode next = null;
                if(curr != null){
                    next = curr.next;   // next用于记录 拆分完两个链表的结束位置
                    curr.next = null;   // 断开连接
                }

                // 3.4 合并两个subLen长度的有序链表
                ListNode merged = mergeTwoLists(head_1,head_2);
                prev.next = merged;        // prev.next 指向排好序链表的头
                while(prev.next != null){  // while循环 将prev移动到 subLen*2 的位置后去
                    prev = prev.next;
                }
                curr = next;              // next用于记录 拆分完两个链表的结束位置
            }
        }
        // 返回新排好序的链表
        return dummyHead.next;
    }


    // 此处是Leetcode21 --> 合并两个有序链表
    public ListNode mergeTwoLists(ListNode l1,ListNode l2){
        ListNode dummy = new ListNode(0);
        ListNode curr  = dummy;

        while(l1 != null && l2!= null){ // 退出循环的条件是走完了其中一个链表
            // 判断l1 和 l2大小
            if (l1.val < l2.val){
                // l1 小 ， curr指向l1
                curr.next = l1;
                l1 = l1.next;       // l1 向后走一位
            }else{
                // l2 小 ， curr指向l2
                curr.next = l2;
                l2 = l2.next;       // l2向后走一位
            }
            curr = curr.next;       // curr后移一位
        }

        // 退出while循环之后,比较哪个链表剩下长度更长,直接拼接在排序链表末尾
        if(l1 == null) curr.next = l2;
        if(l2 == null) curr.next = l1;

        // 最后返回合并后有序的链表
        return dummy.next;
    }
}
```

- 时间复杂度O(nlogn)

- 空间复杂度O(1)

方法二：自顶向下归并排序

![Picture2.png](https://pic.leetcode-cn.com/8c47e58b6247676f3ef14e617a4686bc258cc573e36fcf67c1b0712fa7ed1699-Picture2.png)

```java
class Solution {
    public ListNode sortList(ListNode head) {
        return sortList(head, null);
    }

    public ListNode sortList(ListNode head, ListNode tail) {
        if (head == null) {
            return head;
        }
        if (head.next == tail) {
            head.next = null;
            return head;
        }
        ListNode slow = head, fast = head;
        while (fast != tail) {
            slow = slow.next;
            fast = fast.next;
            if (fast != tail) {
                fast = fast.next;
            }
        }
        ListNode mid = slow;
        ListNode list1 = sortList(head, mid);
        ListNode list2 = sortList(mid, tail);
        ListNode sorted = merge(list1, list2);
        return sorted;
    }

    public ListNode merge(ListNode head1, ListNode head2) {
        ListNode dummyHead = new ListNode(0);
        ListNode temp = dummyHead, temp1 = head1, temp2 = head2;
        while (temp1 != null && temp2 != null) {
            if (temp1.val <= temp2.val) {
                temp.next = temp1;
                temp1 = temp1.next;
            } else {
                temp.next = temp2;
                temp2 = temp2.next;
            }
            temp = temp.next;
        }
        if (temp1 != null) {
            temp.next = temp1;
        } else if (temp2 != null) {
            temp.next = temp2;
        }
        return dummyHead.next;
    }
}
```

- 时间复杂度O(nlogn)

- 空间复杂度O(logn)

### [016] 寻找重复数

给定一个包含 n + 1 个整数的数组 nums，其数字都在 1 到 n 之间（包括 1 和 n），可知至少存在一个重复的整数。假设只有一个重复的整数，找出这个重复的数。

示例 1:

```
输入: [1,3,4,2,2]
输出: 2
```

示例 2:

```java
输入: [3,1,3,4,2]
输出: 3
```

说明：

- 不能更改原数组（假设数组是只读的）。
- 只能使用额外的 O(1) 的空间。
- 时间复杂度小于 O(n^2) 。
- 数组中只有一个重复的数字，但它可能不止重复出现一次。

方法一：原地置换（题目不允许修改原数组）

遍历交换元素到其索引位置（值1->index0 值2->index1 ...）

```java
class Solution {
    public int findDuplicate(int[] nums) {
        int temp;
        for (int i = 0; i < nums.length; i++) {
            while (nums[i]-1 != i){
                //索引 与 索引对应的值 相等， 该数字已经存在了
                if(nums[i] == nums[nums[i] -1]){
                    return nums[i];
                }
                temp=nums[i];
                nums[i]=nums[temp -1];
                nums[temp-1]=temp;
            }
        }
        return -1;
    }
}
```

- 时间复杂度：O(n)

- 空间复杂度：O(1)

方法二：快慢指针

「Floyd 判圈算法」（又称龟兔赛跑算法），检测链表是否有环的算法

先设置慢指针 slow 和快指针 fast ，慢指针每次走一步，快指针每次走两步，根据「Floyd 判圈算法」两个指针在有环的情况下一定会相遇，此时我们再将 slow 放置起点 0，两个指针每次同时移动一步，相遇的点就是答案。

```java
class Solution {
    public int findDuplicate(int[] nums) {
        int slow = 0, fast = 0;
        do {
            slow = nums[slow];
            fast = nums[nums[fast]];
        } while (slow != fast);//相遇点
        slow = 0;
        while (slow != fast) {
            slow = nums[slow];
            fast = nums[fast];
        }//环点
        return slow;
    }
}

```

- 时间复杂度：O(n)。「Floyd 判圈算法」时间复杂度为线性的时间复杂度。
- 空间复杂度：O(1)。我们只需要常数空间存放若干变量。

方法三：二分查找

```java
class Solution {
    public int findDuplicate(int[] nums) {
        int n = nums.length;
        int l = 1, r = n - 1, ans = -1;
        while (l <= r) {
            int mid = (l + r) >> 1;
            int cnt = 0;
            for (int i = 0; i < n; ++i) {
                if (nums[i] <= mid) {
                    cnt++;
                }
            }
            if (cnt <= mid) {
                l = mid + 1;
            } else {
                r = mid - 1;
                ans = mid;
            }
        }
        return ans;
    }
}
```

- 时间复杂度：O(nlogn)，其中 n 为 nums[] 数组的长度。二分查找最多需要二分 O(logn) 次，每次判断的时候需要O(n) 遍历 nums[] 数组求解小于等于 mid 的数的个数，因此总时间复杂度为 O(nlogn)。
- 空间复杂度：O(1)。我们只需要常数空间存放若干变量。

方法四：二进制

```java
class Solution {
    public int findDuplicate(int[] nums) {
        int n = nums.length, ans = 0;
        int bit_max = 31;
        while (((n - 1) >> bit_max) == 0) {
            bit_max -= 1;
        }
        for (int bit = 0; bit <= bit_max; ++bit) {
            int x = 0, y = 0;
            for (int i = 0; i < n; ++i) {
                if ((nums[i] & (1 << bit)) != 0) {
                    x += 1;
                }
                if (i >= 1 && ((i & (1 << bit)) != 0)) {
                    y += 1;
                }
            }
            if (x > y) {
                ans |= 1 << bit;
            }
        }
        return ans;
    }
}
```

- 时间复杂度：O(nlog n)

- 空间复杂度：O(1)

### [017] 二叉树的最近公共祖先

给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

百度百科中最近公共祖先的定义为：“对于有根树 T 的两个结点 p、q，最近公共祖先表示为一个结点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（一个节点也可以是它自己的祖先）。”

例如，给定如下二叉树:  root = [3,5,1,6,2,0,8,null,null,7,4]

方法一：

> 同 剑值offer 简单部分 [013] 二叉树的最近公共祖先

```java
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if (root == null) return null;
        if (root.val == p.val) return root;
        if (root.val == q.val) return root;
        TreeNode left = lowestCommonAncestor(root.left,p,q);
        TreeNode right = lowestCommonAncestor(root.right,p,q);
        if (left == null) return right;
        if (right == null) return left;
        return root;
    }
}
```

- 时间复杂度 O(N) ： 其中 N 为二叉树节点数；最差情况下，需要递归遍历树的所有节点。

- 空间复杂度 O(N) ： 最差情况下，递归深度达到 N ，系统使用 O(N) 大小的额外空间。

### [018] 每日温度

请根据每日 气温 列表，重新生成一个列表。对应位置的输出为：要想观测到更高的气温，至少需要等待的天数。如果气温在这之后都不会升高，请在该位置用 0 来代替。

例如，给定一个列表 `temperatures = [73, 74, 75, 71, 69, 72, 76, 73]`，你的输出应该是 `[1, 1, 4, 2, 1, 1, 0, 0]`。

提示：气温 列表长度的范围是 `[1, 30000]`。每个气温的值的均为华氏度，都是在 `[30, 100]` 范围内的整数。

方法一：暴力

```java
public class Solution {
    public int[] dailyTemperatures(int[] T) {
        int[] ret = new int[T.length];
        for (int i = 0; i < T.length; i++) {
            for (int j = i +1; j < T.length; j++) {
                if (T[j] > T[i]) {
                    ret[i] = j - i;
                    break;
                }
            }
        }
        return ret;
    }
}
```

- 时间复杂度 O(N^2)
- 空间复杂度 O(1)

方法二：暴力优化，减少遍历次数

```java
class Solution {
    public int[] dailyTemperatures(int[] T) {
        int[] result = new int[T.length];
        //从右向左遍历
        for (int i = T.length - 2; i >= 0; i--) {
            // j+= result[j]是利用已经有的结果进行跳跃
            for (int j = i + 1; j < T.length; j+= result[j]) {
                if (T[j] > T[i]) {
                    result[i] = j - i;
                    break;
                } else if (result[j] == 0) { //遇到0表示后面不会有更大的值，那当然当前值就应该也为0
                    result[i] = 0;
                    break;
                }
            }
        }
        return result;
    }
}
```

- 时间复杂度 O(N^2)
- 空间复杂度 O(1)

方法三：栈

维护一个存储下标的单调栈，从栈底到栈顶的下标对应的温度列表中的温度依次递减。如果一个下标在单调栈里，则表示尚未找到下一次温度更高的下标。

正向遍历温度列表。对于温度列表中的每个元素 T[i]：

- 如果栈为空，则直接将 i 进栈，

- 如果栈不为空，则比较栈顶元素 prevIndex 对应的温度 `T[prevIndex]` 和当前温度 `T[i]`，
  - 如果 `T[i] > T[prevIndex]`，则将 prevIndex 移除，并将 prevIndex 对应的等待天数赋为 `i - prevIndex`，
  - 重复上述操作直到栈为空或者栈顶元素对应的温度小于等于当前温度，然后将 i 进栈。

![739.gif](https://pic.leetcode-cn.com/7a133e857271e638c04b3a27c1eabc29570e585cc44d7da60eb039459a7f89cd-739.gif)

```java
class Solution {
    public int[] dailyTemperatures(int[] T) {
        int length = T.length;
        int[] ans = new int[length];
        Deque<Integer> stack = new LinkedList<Integer>();
        for (int i = 0; i < length; i++) {
            int temperature = T[i];
            while (!stack.isEmpty() && temperature > T[stack.peek()]) {
                int prevIndex = stack.pop();
                ans[prevIndex] = i - prevIndex;
            }
            stack.push(i);
        }
        return ans;
    }
}
```

- 时间复杂度 O(N)
- 空间复杂度 O(N)

### [019] 字母异位词分组

给定一个字符串数组，将字母异位词组合在一起。字母异位词指字母相同，但排列不同的字符串。

示例:

```java
输入: ["eat", "tea", "tan", "ate", "nat", "bat"]
输出:
[
  ["ate","eat","tea"],
  ["nat","tan"],
  ["bat"]
]
```

说明：

- 所有输入均为小写字母。
- 不考虑答案输出的顺序。

方法一：排序

互为字母异位词的两个字符串包含的字母相同，因此对两个字符串分别进行排序之后得到的字符串一定是相同的，故可以将排序之后的字符串作为哈希表的键。

```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        Map<String, List<String>> map = new HashMap<String, List<String>>();
        for (String str : strs) {
            char[] array = str.toCharArray();
            Arrays.sort(array);
            String key = new String(array);
            List<String> list = map.getOrDefault(key, new ArrayList<String>());
            list.add(str);
            map.put(key, list);
        }
        return new ArrayList<List<String>>(map.values());
    }
}
```

- 时间复杂度：O(nklogk)，其中 n 是strs 中的字符串的数量，k 是 strs 中的字符串的的最大长度。需要遍历 n 个字符串，对于每个字符串，需要 O(klogk) 的时间进行排序以及 O(1) 的时间更新哈希表，因此总时间复杂度是 O(nklogk)。

- 空间复杂度：O(nk)，其中 n 是 strs 中的字符串的数量，k 是strs 中的字符串的的最大长度。需要用哈希表存储全部字符串。

方法二：计数

由于互为字母异位词的两个字符串包含的字母相同，因此两个字符串中的相同字母出现的次数一定是相同的，故可以将每个字母出现的次数使用字符串表示，作为哈希表的键。

```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        Map<String, List<String>> map = new HashMap<String, List<String>>();
        for (String str : strs) {
            int[] counts = new int[26];
            int length = str.length();
            for (int i = 0; i < length; i++) {
                counts[str.charAt(i) - 'a']++;
            }
            // 将每个出现次数大于 0 的字母和出现次数按顺序拼接成字符串，作为哈希表的键
            StringBuffer sb = new StringBuffer();
            for (int i = 0; i < 26; i++) {
                if (counts[i] != 0) {
                    sb.append((char) ('a' + i));
                    sb.append(counts[i]);
                }
            }
            String key = sb.toString();
            List<String> list = map.getOrDefault(key, new ArrayList<String>());
            list.add(str);
            map.put(key, list);
        }
        return new ArrayList<List<String>>(map.values());
    }
}
```

时间复杂度：O(n(k+∣Σ∣))，其中 n 是 strs 中的字符串的数量，k 是 strs 中的字符串的的最大长度，Σ 是字符集，在本题中字符集为所有小写字母，∣Σ∣=26。需要遍历 n 个字符串，对于每个字符串，需要 O(k) 的时间计算每个字母出现的次数，O(∣Σ∣) 的时间生成哈希表的键，以及 O(1) 的时间更新哈希表，因此总时间复杂度是 O(n(k+∣Σ∣))。

空间复杂度：O(n(k+∣Σ∣))，其中 n 是strs 中的字符串的数量，k 是 strs 中的字符串的最大长度，Σ 是字符集，在本题中字符集为所有小写字母，∣Σ∣=26。需要用哈希表存储全部字符串，而记录每个字符串中每个字母出现次数的数组需要的空间为 O(∣Σ∣)，在渐进意义下小于 O(n(k+∣Σ∣))，可以忽略不计。

### [020] 把二叉搜索树转换为累加树

给出二叉 搜索 树的根节点，该树的节点值各不相同，请你将其转换为累加树（Greater Sum Tree），使每个节点 node 的新值等于原树中大于或等于 node.val 的值之和。

提醒一下，二叉搜索树满足下列约束条件：

- 节点的左子树仅包含键 小于 节点键的节点。
- 节点的右子树仅包含键 大于 节点键的节点。
- 左右子树也必须是二叉搜索树。

示例 1：

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2019/05/03/tree.png)

输入：[4,1,6,0,2,5,7,null,null,null,3,null,null,null,8]
输出：[30,36,21,36,35,26,15,null,null,null,33,null,null,null,8]

**提示：**

- 树中的节点数介于 `0` 和 `104` 之间。
- 每个节点的值介于 `-104` 和 `104` 之间。
- 树中的所有值 **互不相同** 。
- 给定的树为二叉搜索树。



方法一·：反序中序遍历

二叉搜索树的中序遍历是一个单调递增的有序序列。如果我们反序地中序遍历该二叉搜索树，即可得到一个单调递减的有序序列。

```java
class Solution {
    int sum = 0;
    public TreeNode convertBST(TreeNode root) {
        if (root != null) {
            convertBST(root.right);
            sum += root.val;
            root.val = sum;
            convertBST(root.left);
        }
        return root;
    }
}
```

时间复杂度：O(n)，其中 n 是二叉搜索树的节点数。每一个节点恰好被遍历一次。

空间复杂度：O(n)，为递归过程中栈的开销，平均情况下为 O(logn)，最坏情况下树呈现链状，为 O(n)。

方法二：Morris 遍历

Morris 遍历的核心思想是利用树的大量空闲指针，实现空间开销的极限缩减。其反序中序遍历规则总结如下：

- 如果当前节点的右子节点为空，处理当前节点，并遍历当前节点的左子节点；

- 如果当前节点的右子节点不为空，找到当前节点右子树的最左节点（该节点为当前节点中序遍历的前驱节点）；
  - 如果最左节点的左指针为空，将最左节点的左指针指向当前节点，遍历当前节点的右子节点；
  - 如果最左节点的左指针不为空，将最左节点的左指针重新置为空（恢复树的原状），处理当前节点，并将当前节点置为其左节点；

- 重复上述步骤，直到遍历结束。

```java
class Solution {
    public TreeNode convertBST(TreeNode root) {
        int sum = 0;
        TreeNode node = root;

        while (node != null) {
            if (node.right == null) {
                sum += node.val;
                node.val = sum;
                node = node.left;
            } else {
                TreeNode succ = getSuccessor(node);
                if (succ.left == null) {
                    succ.left = node;
                    node = node.right;
                } else {
                    succ.left = null;
                    sum += node.val;
                    node.val = sum;
                    node = node.left;
                }
            }
        }

        return root;
    }

    public TreeNode getSuccessor(TreeNode node) {
        TreeNode succ = node.right;
        while (succ.left != null && succ.left != node) {
            succ = succ.left;
        }
        return succ;
    }
}
```

- 时间复杂度：O(n)，其中 n 是二叉搜索树的节点数。没有左子树的节点只被访问一次，有左子树的节点被访问两次。

- 空间复杂度：O(1)。只操作已经存在的指针（树的空闲指针），因此只需要常数的额外空间。

### [021] 回文子串

给定一个字符串，你的任务是计算这个字符串中有多少个回文子串。

具有不同开始位置或结束位置的子串，即使是由相同的字符组成，也会被视作不同的子串。

示例 1：

```java
输入："abc"
输出：3
解释：三个回文子串: "a", "b", "c"
```

示例 2：

```java
输入："aaa"
输出：6
解释：6个回文子串: "a", "a", "a", "aa", "aa", "aaa"
```


提示：

- 输入的字符串长度不会超过 1000 。

方法一：动态规划法

状态：`dp[i][j]` 表示字符串s在`[i,j]`区间的子串是否是一个回文串。
状态转移方程：当 `s[i] == s[j] && (j - i < 2 || dp[i + 1][j - 1])` 时，`dp[i][j]=true`，否则为false

- 当只有一个字符时，比如 a 自然是一个回文串。
- 当有两个字符时，如果是相等的，比如 a，也是一个回文串。
- 当有三个及以上字符时，比如 `ababa` 这个字符记作串 1，把两边的 a 去掉，也就是 `bab` 记作串 2，可以看出只要串2是一个回文串，那么左右各多了一个 a 的串 1 必定也是回文串。所以当 `s[i]==s[j]` 时，自然要看 `dp[i+1][j-1]` 是不是一个回文串。

```java
class Solution {
    public int countSubstrings(String s) {
       // 动态规划法
        boolean[][] dp = new boolean[s.length()][s.length()];
        int ans = 0;

        for (int j = 0; j < s.length(); j++) {
            for (int i = 0; i <= j; i++) {
                if (s.charAt(i) == s.charAt(j) && (j - i < 2 || dp[i + 1][j - 1])) {
                    dp[i][j] = true;
                    ans++;
                }
            }
        }
        return ans;
    }
}
```

- 时间复杂度为 O(N^2)，

- 空间复杂度为 O(N^2)。

方法二：中心扩展法

```java
class Solution {
   public int countSubstrings(String s) {
        // 中心扩展法
        int ans = 0;
        // 2 * len - 1 个中心点
        // 中心点不能只有单个字符构成，还要包括两个字符
        // 3 个可以由 1 个扩展一次得到，4 个可以由两个扩展一次得到
        for (int center = 0; center < 2 * s.length() - 1; center++) {
            int left = center / 2;
            int right = left + center % 2;

            while (left >= 0 && right < s.length() && s.charAt(left) == s.charAt(right)) {
                ans++;
                left--;
                right++;
            }
        }
        return ans;
    }
}
```

- 时间复杂度是 O(N^2)

- 空间复杂度是 O(1)

### [022] 盛最多水的容器

给你 n 个非负整数 a1，a2，...，an，每个数代表坐标中的一个点 (i, ai) 。在坐标内画 n 条垂直线，垂直线 i 的两个端点分别为 (i, ai) 和 (i, 0) 。找出其中的两条线，使得它们与 x 轴共同构成的容器可以容纳最多的水。

说明：你不能倾斜容器。

示例 1：

![img](https://aliyun-lc-upload.oss-cn-hangzhou.aliyuncs.com/aliyun-lc-upload/uploads/2018/07/25/question_11.jpg)

```
输入：[1,8,6,2,5,4,8,3,7]
输出：49 
解释：图中垂直线代表输入数组 [1,8,6,2,5,4,8,3,7]。在此情况下，容器能够容纳水（表示为蓝色部分）的最大值为 49。
```

示例 2：

```
输入：height = [1,1]
输出：1
```

示例 3：

```
输入：height = [4,3,2,1,4]
输出：16
```

示例 4：

```
输入：height = [1,2,1]
输出：2
```


提示：

- n = height.length
- 2 <= n <= 3 * 10^4
- 0 <= height[i] <= 3 * 10^4

方法一·： 双指针法

算法流程：设置双指针`i,j` 分别位于容器壁两端，根据规则移动指针，并且更新面积最大值 `res`，直到 `i == j` 时返回 `res`。

```java
class Solution {
    public int maxArea(int[] height) {
        int i = 0, j = height.length - 1, res = 0;
        while(i < j){
            res = height[i] < height[j] ? 
                Math.max(res, (j - i) * height[i++]): 
                Math.max(res, (j - i) * height[j--]); 
        }
        return res;
    }
}
```

- 时间复杂度 O(N)，双指针遍历一次底边宽度 N 。
- 空间复杂度 O(1)，指针使用常数额外空间。

### [023] 数组中的第K个最大元素

在未排序的数组中找到第 k 个最大的元素。请注意，你需要找的是数组排序后的第 k 个最大的元素，而不是第 k 个不同的元素。

示例 1:

```
输入: [3,2,1,5,6,4] 和 k = 2
输出: 5
```

示例 2:

```
输入: [3,2,3,1,2,4,5,5,6] 和 k = 4
输出: 4
```

说明:

你可以假设 k 总是有效的，且 1 ≤ k ≤ 数组的长度。

方法一：暴力

```java
class Solution {
    public int findKthLargest(int[] nums, int k) {
        Arrays.sort(nums);
        return nums[nums.length-k];
    }
}
```

- 时间复杂度：O(NlogN)，这里 N 是数组的长度，JDK 默认使用快速排序，因此时间复杂度为 O(NlogN)。
- 空间复杂度：O(1)，这里是原地排序，没有借助额外的辅助空间。 

方法二：堆排序

小根堆

```java
public class Solution {

    public int findKthLargest(int[] nums, int k) {
        int len = nums.length;
        // 使用一个含有 len 个元素的最小堆，默认是最小堆，可以不写 lambda 表达式：(a, b) -> a - b
        PriorityQueue<Integer> minHeap = new PriorityQueue<>(len, (a, b) -> a - b);
        for (int i = 0; i < len; i++) {
            minHeap.add(nums[i]);
        }
        for (int i = 0; i < len - k; i++) {
            minHeap.poll();
        }
        return minHeap.peek();
    }
}
```

大根堆

```java
public class Solution {

    public int findKthLargest(int[] nums, int k) {
        int len = nums.length;
        // 使用一个含有 len 个元素的最大堆，lambda 表达式应写成：(a, b) -> b - a
        PriorityQueue<Integer> minHeap = new PriorityQueue<>(len, (a, b) -> b - a);
        for (int i = 0; i < len; i++) {
            minHeap.add(nums[i]);
        }
        for (int i = 0; i < k -1 ; i++) {
            minHeap.poll();
        }
        return minHeap.peek();
    }
}
```

只用 `k` 个容量的优先队列，而不用全部 `len` 个容量。

```java
public class Solution {

    public int findKthLargest(int[] nums, int k) {
        int len = nums.length;
        // 使用一个含有 k 个元素的最小堆
        PriorityQueue<Integer> minHeap = new PriorityQueue<>(k, (a, b) -> a - b);
        for (int i = 0; i < k; i++) {
            minHeap.add(nums[i]);
        }
        for (int i = k; i < len; i++) {
            // 看一眼，不拿出，因为有可能没有必要替换
            Integer topEle = minHeap.peek();
            // 只要当前遍历的元素比堆顶元素大，堆顶弹出，遍历的元素进去
            if (nums[i] > topEle) {
                minHeap.poll();
                minHeap.add(nums[i]);
            }
        }
        return minHeap.peek();
    }
}
```

用 `k + 1` 个容量的优先队列，使得上面的过程更“连贯”一些，到了 `k` 个以后的元素，就进来一个，出去一个，让优先队列自己去维护大小关系。

```java
public class Solution {

    public int findKthLargest(int[] nums, int k) {
        int len = nums.length;
        // 最小堆
        PriorityQueue<Integer> priorityQueue = new PriorityQueue<>(k + 1, (a, b) -> (a - b));
        for (int i = 0; i < k; i++) {
            priorityQueue.add(nums[i]);
        }
        for (int i = k; i < len; i++) {
            priorityQueue.add(nums[i]);
            priorityQueue.poll();
        }
        return priorityQueue.peek();
    }
}
```

综合考虑以上两种情况，总之都是为了节约空间复杂度。即 `k` 较小的时候使用最小堆，`k` 较大的时候使用最大堆。

```java
public class Solution {

    // 根据 k 的不同，选最大堆和最小堆，目的是让堆中的元素更小
    // 思路 1：k 要是更靠近 0 的话，此时 k 是一个较小的数，用最大堆
    // 例如在一个有 6 个元素的数组里找第 5 大的元素
    // 思路 2：k 要是更靠近 len 的话，用最小堆

    // 所以分界点就是 k = len - k

    public int findKthLargest(int[] nums, int k) {
        int len = nums.length;
        if (k <= len - k) {
            // System.out.println("使用最小堆");
            // 特例：k = 1，用容量为 k 的最小堆
            // 使用一个含有 k 个元素的最小堆
            PriorityQueue<Integer> minHeap = new PriorityQueue<>(k, (a, b) -> a - b);
            for (int i = 0; i < k; i++) {
                minHeap.add(nums[i]);
            }
            for (int i = k; i < len; i++) {
                // 看一眼，不拿出，因为有可能没有必要替换
                Integer topEle = minHeap.peek();
                // 只要当前遍历的元素比堆顶元素大，堆顶弹出，遍历的元素进去
                if (nums[i] > topEle) {
                    minHeap.poll();
                    minHeap.add(nums[i]);
                }
            }
            return minHeap.peek();

        } else {
            // System.out.println("使用最大堆");
            assert k > len - k;
            // 特例：k = 100，用容量为 len - k + 1 的最大堆
            int capacity = len - k + 1;
            PriorityQueue<Integer> maxHeap = new PriorityQueue<>(capacity, (a, b) -> b - a);
            for (int i = 0; i < capacity; i++) {
                maxHeap.add(nums[i]);
            }
            for (int i = capacity; i < len; i++) {
                // 看一眼，不拿出，因为有可能没有必要替换
                Integer topEle = maxHeap.peek();
                // 只要当前遍历的元素比堆顶元素大，堆顶弹出，遍历的元素进去
                if (nums[i] < topEle) {
                    maxHeap.poll();
                    maxHeap.add(nums[i]);
                }
            }
            return maxHeap.peek();
        }
    }
}
```

- 时间复杂度 O(NlogK)

- 空间复杂度 O(N)

方法三：快排思想

```java
public class Solution {

    public int findKthLargest(int[] nums, int k) {
        int len = nums.length;
        int left = 0;
        int right = len - 1;

        // 转换一下，第 k 大元素的索引是 len - k
        int target = len - k;

        while (true) {
            int index = partition(nums, left, right);
            if (index == target) {
                return nums[index];
            } else if (index < target) {
                left = index + 1;
            } else {
                right = index - 1;
            }
        }
    }

    /**
     * 在数组 nums 的子区间 [left, right] 执行 partition 操作，返回 nums[left] 排序以后应该在的位置
     * 在遍历过程中保持循环不变量的语义
     * 1、[left + 1, j] < nums[left]
     * 2、(j, i] >= nums[left]
     */
    public int partition(int[] nums, int left, int right) {
        int pivot = nums[left];
        int j = left;
        for (int i = left + 1; i <= right; i++) {
            if (nums[i] < pivot) {
                // 小于 pivot 的元素都被交换到前面
                j++;
                swap(nums, j, i);
            }
        }
        // 在之前遍历的过程中，满足 [left + 1, j] < pivot，并且 (j, i] >= pivot
        swap(nums, j, left);
        // 交换以后 [left, j - 1] < pivot, nums[j] = pivot, [j + 1, right] >= pivot
        return j;
    }

    private void swap(int[] nums, int index1, int index2) {
        int temp = nums[index1];
        nums[index1] = nums[index2];
        nums[index2] = temp;
    }
}
```

> **注意：本题必须随机初始化 `pivot` 元素，否则通过时间会很慢，因为测试用例中有极端测试用例。**

为了应对极端测试用例，使得递归树加深，可以在循环一开始的时候，随机交换第 1 个元素与它后面的任意 1 个元素的位置；

说明：最极端的是顺序数组与倒序数组，此时递归树画出来是链表，时间复杂度是 O(N^2)，根本达不到减治的效果。

```java
public class Solution {
    private static Random random = new Random(System.currentTimeMillis());

    public int findKthLargest(int[] nums, int k) {
        int len = nums.length;
        int target = len - k;
        int left = 0;
        int right = len - 1;
        while (true) {
            int index = partition(nums, left, right);
            if (index < target) {
                left = index + 1;
            } else if (index > target) {
                right = index - 1;
            } else {
                return nums[index];
            }
        }
    }

    // 在区间 [left, right] 这个区间执行 partition 操作

    private int partition(int[] nums, int left, int right) {
        // 在区间随机选择一个元素作为标定点
        if (right > left) {
            int randomIndex = left + 1 + random.nextInt(right - left);
            swap(nums, left, randomIndex);
        }

        int pivot = nums[left];
        int j = left;
        for (int i = left + 1; i <= right; i++) {
            if (nums[i] < pivot) {
                j++;
                swap(nums, j, i);
            }
        }
        swap(nums, left, j);
        return j;
    }

    private void swap(int[] nums, int index1, int index2) {
        int temp = nums[index1];
        nums[index1] = nums[index2];
        nums[index2] = temp;
    }
}
```

使用双指针，将与 `pivot` 相等的元素等概论地分到 `pivot` 最终排定位置的两边。

使用双指针的办法找到切分元素的位置。

```java
public class Solution {
    private static Random random = new Random(System.currentTimeMillis());
    
    public int findKthLargest(int[] nums, int k) {
        int len = nums.length;
        int left = 0;
        int right = len - 1;

        // 转换一下，第 k 大元素的索引是 len - k
        int target = len - k;

        while (true) {
            int index = partition(nums, left, right);
            if (index == target) {
                return nums[index];
            } else if (index < target) {
                left = index + 1;
            } else {
                right = index - 1;
            }
        }
    }

    public int partition(int[] nums, int left, int right) {
        // 在区间随机选择一个元素作为标定点
        if (right > left) {
            int randomIndex = left + 1 + random.nextInt(right - left);
            swap(nums, left, randomIndex);
        }

        int pivot = nums[left];

        // 将等于 pivot 的元素分散到两边
        // [left, lt) <= pivot
        // (rt, right] >= pivot

        int lt = left + 1;
        int rt = right;

        while (true) {
            while (lt <= rt && nums[lt] < pivot) {
                lt++;
            }
            while (lt <= rt && nums[rt] > pivot) {
                rt--;
            }

            if (lt > rt) {
                break;
            }
            swap(nums, lt, rt);
            lt++;
            rt--;
        }

        swap(nums, left, rt);
        return rt;
    }

    private void swap(int[] nums, int index1, int index2) {
        int temp = nums[index1];
        nums[index1] = nums[index2];
        nums[index2] = temp;
    }
}
```

- 时间复杂度：O(N))，这里 N 是数组的长度
- 空间复杂度：O(1)，原地排序，没有借助额外的辅助空间

### [024] 二叉树的层序遍历

给你一个二叉树，请你返回其按 层序遍历 得到的节点值。 （即逐层地，从左到右访问所有节点）。

示例：

二叉树：[3,9,20,null,null,15,7],

    	3
       / \
      9  20
        /  \
       15   7

返回其层序遍历结果：

```
[
  [3],
  [9,20],
  [15,7]
]
```

方法一：广度优先搜索

```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> ret = new ArrayList<List<Integer>>();
        if (root == null) {
            return ret;
        }

        Queue<TreeNode> queue = new LinkedList<TreeNode>();
        queue.offer(root);
        while (!queue.isEmpty()) {
            List<Integer> level = new ArrayList<Integer>();
            int currentLevelSize = queue.size();
            for (int i = 1; i <= currentLevelSize; ++i) {
                TreeNode node = queue.poll();
                level.add(node.val);
                if (node.left != null) {
                    queue.offer(node.left);
                }
                if (node.right != null) {
                    queue.offer(node.right);
                }
            }
            ret.add(level);
        }
        
        return ret;
    }
}
```

- 时间复杂度：每个点进队出队各一次，故渐进时间复杂度为 O(n) 
- 空间复杂度：队列中元素的个数不超过 n*n* 个，故渐进空间复杂度为 O(n) 

### [025] 不同路径

一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为 “Start” ）。

机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish” ）。

问总共有多少条不同的路径？

 

示例 1：

![img](https://assets.leetcode.com/uploads/2018/10/22/robot_maze.png)

```
输入：m = 3, n = 7
输出：28
```

示例 2：

```
输入：m = 3, n = 2
输出：3
解释：
从左上角开始，总共有 3 条路径可以到达右下角。

1. 向右 -> 向右 -> 向下
2. 向右 -> 向下 -> 向右
3. 向下 -> 向右 -> 向右
```

示例 3：

```
输入：m = 7, n = 3
输出：28
```

示例 4：

```
输入：m = 3, n = 3
输出：6
```


提示：

- 1 <= m, n <= 100
- 题目数据保证答案小于等于 2 * 109

方法一：动态规划

我们用 f(i, j) 表示从左上角走到 (i, j) 的路径数量，其中 i 和 j 的范围分别是 [0, m) 和 [0, n)。

每一步只能从向下或者向右移动一步，故
$$
f(i,j)=f(i−1,j)+f(i,j−1)
$$
需要注意的是，如果 i=0，那么 f(i-1,j) 并不是一个满足要求的状态，我们需要忽略这一项；

同理，如果 j=0，那么 f(i,j-1) 并不是一个满足要求的状态，我们需要忽略这一项。

初始条件为 f(0,0)=1，即从左上角走到左上角有一种方法。

最终的答案即为 f(m-1,n-1)。

```java
class Solution {
    public int uniquePaths(int m, int n) {
        int[][] f = new int[m][n];
        for (int i = 0; i < m; ++i) {
            f[i][0] = 1;
        }
        for (int j = 0; j < n; ++j) {
            f[0][j] = 1;
        }
        for (int i = 1; i < m; ++i) {
            for (int j = 1; j < n; ++j) {
                f[i][j] = f[i - 1][j] + f[i][j - 1];
            }
        }
        return f[m - 1][n - 1];
    }
}
```

- 时间复杂度：O(mn)
- 空间复杂度：O(mn)

动态规划优化：当前坐标的值只和左边与上面的值有关，和其他的无关，这样二维数组造成大量的空间浪费，所以我们可以把它改为一维数组。

```java
public int uniquePaths(int m, int n) {
    int[] dp = new int[m];
    Arrays.fill(dp, 1);
    for (int j = 1; j < n; j++)
        for (int i = 1; i < m; i++)
            dp[i] += dp[i - 1];
    return dp[m - 1];
}
```

- 时间复杂度：O(mn)
- 空间复杂度：O(n)

方法二：组合数学

从左上角到右下角的过程中，我们需要移动 m+n-2 次，其中有 m-1 次向下移动，n-1 次向右移动。因此路径的总数，就等于从 m+n-2 次移动中选择 m-1 次向下移动的方案数，即组合数：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201231104107423.png#pic_center)

因此我们直接计算出这个组合数即可。

```java
class Solution {
    public int uniquePaths(int m, int n) {
        long ans = 1;
        for (int x = n, y = 1; y < m; ++x, ++y) {
            ans = ans * x / y;
        }
        return (int) ans;
    }
}
```

- 时间复杂度：O(m)。由于我们交换行列的值并不会对答案产生影响，因此我们总可以通过交换 m 和 n 使得 m≤n，这样空间复杂度降低至 O(min(m,n))。

- 空间复杂度：O(1)。

### [026] 前 K 个高频元素

给定一个非空的整数数组，返回其中出现频率前 k 高的元素。 

示例 1:

```
输入: nums = [1,1,1,2,2,3], k = 2
输出: [1,2]
```

示例 2:

```
输入: nums = [1], k = 1
输出: [1]
```


提示：

- 你可以假设给定的 k 总是合理的，且 1 ≤ k ≤ 数组中不相同的元素的个数。
- 你的算法的时间复杂度必须优于 O(n log n) , n 是数组的大小。
- 题目数据保证答案唯一，换句话说，数组中前 k 个高频元素的集合是唯一的。
- 你可以按任意顺序返回答案。

方法一：堆

首先遍历整个数组，并使用哈希表记录每个数字出现的次数，并形成一个「出现次数数组」。找出原数组的前 k 个高频元素，就相当于找出「出现次数数组」的前 kk 大的值。

最简单的做法是给「出现次数数组」排序。但由于可能有 O(N) 个不同的出现次数（其中 N 为原数组长度），故总的算法复杂度会达到 O(NlogN)，不满足题目的要求。

在这里，我们可以利用堆的思想：建立一个小顶堆，然后遍历「出现次数数组」：

- 如果堆的元素个数小于 k，就可以直接插入堆中。
- 如果堆的元素个数等于 k，则检查堆顶与当前出现次数的大小。如果堆顶更大，说明至少有 k 个数字的出现
- 次数比当前值大，故舍弃当前值；否则，就弹出堆顶，并将当前值插入堆中。
- 遍历完成后，堆中的元素就代表了「出现次数数组」中前 k 大的值。

```java
class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        Map<Integer,Integer> map = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            map.put(nums[i], map.getOrDefault(nums[i],0) + 1);
        }

        // int[] 的第一个元素代表数组的值，第二个元素代表了该值出现的次数
        PriorityQueue<int[]> queue = new PriorityQueue<int[]>((m, n) -> m[1] - n[1]);
        for (Map.Entry<Integer, Integer> entry : map.entrySet()) {
            int num = entry.getKey(), count = entry.getValue();
            if (queue.size() == k) {
                if (queue.peek()[1] < count) {
                    queue.poll();
                    queue.offer(new int[]{num, count});
                }
            } else {
                queue.offer(new int[]{num, count});
            }
        }
        int[] ret = new int[k];
        for (int i = 0; i < k; ++i) {
            ret[i] = queue.poll()[0];
        }
        return ret;
    }
}
```

- 时间复杂度：O(Nlogk)，其中 N 为数组的长度。我们首先遍历原数组，并使用哈希表记录出现次数，每个元素需要 O(1) 的时间，共需 O(N) 的时间。随后，我们遍历「出现次数数组」，由于堆的大小至多为 k，因此每次堆操作需要 O(logk) 的时间，共需 O(Nlogk) 的时间。二者之和为O(Nlogk)。
- 空间复杂度：O(N)。哈希表的大小为 O(N)，而堆的大小为 O(k)，共计为 O(N)。

方法二：基于快速排序

我们可以使用基于快速排序的方法，求出「出现次数数组」的前 k 大的值。

在对数组 `arr[l…r]` 做快速排序的过程中，我们首先将数组划分为两个部分 `arr[i…q−1]` 与 `arr[q+1…j]`，并使得 `arr[i…q−1]` 中的每一个值都不超过 `arr[q]`，且 `arr[q+1…j]` 中的每一个值都大于`arr[q]`。

于是，我们根据 k 与左侧子数组 `arr[i…q−1]` 的长度（为 `q-i`）的大小关系：

- 如果` k≤q−i`，则数组 `arr[l…r]` 前 k 大的值，就等于子数组 `arr[i…q−1]` 前 k 大的值。
- 否则，数组`arr[l…r] `前 k 大的值，就等于左侧子数组全部元素，加上右侧子数组 `arr[q+1…j]` 中前 `k−(q−i)` 大的值。

原版的快速排序算法的平均时间复杂度为 O(NlogN)。我们的算法中，每次只需在其中的一个分支递归即可，因此算法的平均时间复杂度降为 O(N)

```java
class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        Map<Integer, Integer> occurrences = new HashMap<Integer, Integer>();
        for (int num : nums) {
            occurrences.put(num, occurrences.getOrDefault(num, 0) + 1);
        }

        List<int[]> values = new ArrayList<int[]>();
        for (Map.Entry<Integer, Integer> entry : occurrences.entrySet()) {
            int num = entry.getKey(), count = entry.getValue();
            values.add(new int[]{num, count});
        }
        int[] ret = new int[k];
        qsort(values, 0, values.size() - 1, ret, 0, k);
        return ret;
    }

    public void qsort(List<int[]> values, int start, int end, int[] ret, int retIndex, int k) {
        int picked = (int) (Math.random() * (end - start + 1)) + start;
        Collections.swap(values, picked, start);
        
        int pivot = values.get(start)[1];
        int index = start;
        for (int i = start + 1; i <= end; i++) {
            if (values.get(i)[1] >= pivot) {
                Collections.swap(values, index + 1, i);
                index++;
            }
        }
        Collections.swap(values, start, index);

        if (k <= index - start) {
            qsort(values, start, index - 1, ret, retIndex, k);
        } else {
            for (int i = start; i <= index; i++) {
                ret[retIndex++] = values.get(i)[0];
            }
            if (k > index - start + 1) {
                qsort(values, index + 1, end, ret, retIndex, k - (index - start + 1));
            }
        }
    }
}
```

- 时间复杂度：O(N^2)，其中 N 为数组的长度。

  设处理长度为 N 的数组的时间复杂度为 f(N)。由于处理的过程包括一次遍历和一次子分支的递归，最好情况下，有 f(N) = O(N) + f(N/2)，根据 主定理，能够得到 f(N) = O(N)。

  最坏情况下，每次取的中枢数组的元素都位于数组的两端，时间复杂度退化为 O(N^2)。但由于我们在每次递归的开始会先随机选取中枢元素，故出现最坏情况的概率很低。

  平均情况下，时间复杂度为 O(N)。

- 空间复杂度：O(N)。哈希表的大小为 O(N)，用于排序的数组的大小也为 O(N)，快速排序的空间复杂度最好情况为 O(logN)，最坏情况为 O(N)。

### [027] 打家劫舍 III

在上次打劫完一条街道之后和一圈房屋后，小偷又发现了一个新的可行窃的地区。这个地区只有一个入口，我们称之为“根”。 除了“根”之外，每栋房子有且只有一个“父“房子与之相连。一番侦察之后，聪明的小偷意识到“这个地方的所有房屋的排列类似于一棵二叉树”。 如果两个直接相连的房子在同一天晚上被打劫，房屋将自动报警。

计算在不触动警报的情况下，小偷一晚能够盗取的最高金额。

示例 1:

输入: [3,2,3,null,3,null,1]

         3
        / \
       2   3
        \   \ 
         3   1
输出: 7 
解释: 小偷一晚能够盗取的最高金额 = 3 + 3 + 1 = 7.

示例 2:

输入: [3,4,5,1,3,null,1]

         3
        / \
       4   5
      / \   \ 
     1   3   1
输出: 9
解释: 小偷一晚能够盗取的最高金额 = 4 + 5 = 9.

方法一：暴力递归 - 最优子结构

1. 首先要明确相邻的节点不能偷，也就是爷爷选择偷，儿子就不能偷了，但是孙子可以偷
2. 二叉树只有左右两个孩子，一个爷爷最多 2 个儿子，4 个孙子

根据以上条件，我们可以得出单个节点的钱该怎么算

**4 个孙子偷的钱 + 爷爷的钱 VS 两个儿子偷的钱 哪个组合钱多，就当做当前节点能偷的最大钱数。** 这就是动态规划里面的最优子结构

4 个孙子投的钱加上爷爷的钱如下:

`int method1 = root.val + rob(root.left.left) + rob(root.left.right) + rob(root.right.left) + rob(root.right.right)`

两个儿子偷的钱如下:

`int method2 = rob(root.left) + rob(root.right);`

挑选一个钱数多的方案则:

`int result = Math.max(method1, method2);`

```java
class Solution {
    public int rob(TreeNode root) {
        if (root == null) return 0;
        int money = root.val;
        if (root.left != null) {
            money += (rob(root.left.left) + rob(root.left.right));
        }
        if (root.right != null) {
            money += (rob(root.right.left) + rob(root.right.right));
        }
        return Math.max(money, rob(root.left) + rob(root.right));
    }
}
```

方法二：记忆化 - 解决重复子问题

动态规划的关键优化点：重复子问题

由于二叉树不适合拿数组当缓存，我们这次使用哈希表来存储结果，TreeNode 当做 key，能偷的钱当做 value

```java
class Solution {
    public int rob(TreeNode root) {
        HashMap<TreeNode, Integer> memo = new HashMap<>();
        return robInternal(root, memo);
    }

    public int robInternal(TreeNode root, HashMap<TreeNode, Integer> memo) {
        if (root == null) return 0;
        if (memo.containsKey(root)) return memo.get(root);
        int money = root.val;

        if (root.left != null) {
            money += (robInternal(root.left.left, memo) + robInternal(root.left.right, memo));
        }
        if (root.right != null) {
            money += (robInternal(root.right.left, memo) + robInternal(root.right.right, memo));
        }
        int result = Math.max(money, robInternal(root.left, memo) + robInternal(root.right, memo));
        memo.put(root, result);
        return result;
    }
}
```

时间复杂度：O(n)。
空间复杂度：O(n)。

方法三：

计算爷爷节点能偷的钱还要同时去计算孙子节点投的钱，虽然有了记忆化，但是还是有性能损耗。

我们换一种办法来定义此问题

每个节点可选择偷或者不偷两种状态，根据题目意思，相连节点不能一起偷

- 当前节点选择偷时，那么两个孩子节点就不能选择偷了
- 当前节点选择不偷时，两个孩子节点只需要拿最多的钱出来就行(两个孩子节点偷不偷没关系)

我们使用一个大小为 2 的数组来表示偷到的钱 `int[] res = new int[2]` 0 代表不偷，1 代表偷

任何一个节点能偷到的最大钱的状态可以定义为

- 当前节点选择不偷：当前节点能偷到的最大钱数 = 左孩子能偷到的钱 + 右孩子能偷到的钱
- 当前节点选择偷：当前节点能偷到的最大钱数 = 左孩子选择自己不偷时能得到的钱 + 右孩子选择不偷时能得到的钱 + 当前节点的钱数

表示为公式如下

```java
root[0] = Math.max(rob(root.left)[0], rob(root.left)[1]) + Math.max(rob(root.right)[0], rob(root.right)[1])
root[1] = rob(root.left)[0] + rob(root.right)[0] + root.val;
```

```java
class Solution {
    public int rob(TreeNode root) {
        int[] result = robInternal(root);
        return Math.max(result[0], result[1]);
    }

    public int[] robInternal(TreeNode root) {
        if (root == null) return new int[2];
        int[] result = new int[2];

        int[] left = robInternal(root.left);
        int[] right = robInternal(root.right);

        result[0] = Math.max(left[0], left[1]) + Math.max(right[0], right[1]);
        result[1] = left[0] + right[0] + root.val;

        return result;
    }
}
```

- 时间复杂度：O(n)。
- 空间复杂度：O(n)。虽然优化过的版本省去了哈希映射的空间，但是栈空间的使用代价依旧是 O(n)，故空间复杂度不变。

### [028] 完全平方数

给定正整数 n，找到若干个完全平方数（比如 1, 4, 9, 16, ...）使得它们的和等于 n。你需要让组成和的完全平方数的个数最少。

示例 1:

```
输入: n = 12
输出: 3 
解释: 12 = 4 + 4 + 4.
```

示例 2:

```
输入: n = 13
输出: 2
解释: 13 = 4 + 9.
```

方法一：动态规划

dp[i]：表示完全平方数和为i的最小个数。

首先初始化长度为 n+1 的数组 dp，每个位置都为 0。如果 n 为 0，则结果为 0。

对数组进行遍历，下标为 i，每次都将当前数字先更新为最大的结果，即 `dp[i]=i`，比如 `i=4`，最坏结果为 `4=1+1+1+1` 即为 4 个数字。

动态转移方程为：`dp[i] = min(dp[i], dp[i - j * j] + 1)`，i 表示当前数字，`j*j` 表示平方数

意思就是：完全平方数和为i的 最小个数 等于 当前完全平方数和为i的 最大个数与 （完全平方数和为 `i - j * j`的 最小个数 + 完全平方数和为`j * j`的 最小个数）可以看到 `dp[j*j]` 是等于1的

```java
class Solution {
    public int numSquares(int n) {
        int[] dp = new int[n + 1]; // 默认初始化值都为0
        for (int i = 1; i <= n; i++) {
            dp[i] = i; // 最坏的情况就是每次i
            for (int j = 1; i - j * j >= 0; j++) { 
                dp[i] = Math.min(dp[i], dp[i - j * j] + 1); // 动态转移方程
            }
        }
        return dp[n];
    }
}
```

- 时间复杂度：O(n∗sqrt(n))，sqrt 为平方根
- 空间复杂度：O(n)

方法二：回溯法

去考虑所有的分解方案，把过程中的解利用 `HashMap` 全部保存起来，找出最小的解

```java
public int numSquares(int n) {
    return numSquaresHelper(n, new HashMap<Integer, Integer>());
}

private int numSquaresHelper(int n, HashMap<Integer, Integer> map) {
    if (map.containsKey(n)) {
        return map.get(n);
    }
    if (n == 0) {
        return 0;
    }
    int count = Integer.MAX_VALUE;
    for (int i = 1; i * i <= n; i++) {
        count = Math.min(count, numSquaresHelper(n - i * i, map) + 1);
    }
    map.put(n, count);
    return count;
}
```

- 时间复杂度：O(n∗sqrt(n))，sqrt 为平方根
- 空间复杂度：O(n)

### [029] 最佳买卖股票时机含冷冻期

给定一个整数数组，其中第 i 个元素代表了第 i 天的股票价格 。

设计一个算法计算出最大利润。在满足以下约束条件下，你可以尽可能地完成更多的交易（多次买卖一支股票）:

你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。
卖出股票后，你无法在第二天买入股票 (即冷冻期为 1 天)。
示例:

```
输入: [1,2,3,0,2]
输出: 3 
解释: 对应的交易状态为: [买入, 卖出, 冷冻期, 买入, 卖出]
```

方法一：动态规划

`f[i]` 表示第 i 天结束之后的「累计最大收益」。根据题目描述，由于我们最多只能同时买入（持有）一支股票，并且卖出股票后有冷冻期的限制，因此我们会有三种不同的状态：

我们目前持有一支股票，对应的「累计最大收益」记为 `f[i][0]`；

我们目前不持有任何股票，并且处于冷冻期中，对应的「累计最大收益」记为 `f[i][1]`；

我们目前不持有任何股票，并且不处于冷冻期中，对应的「累计最大收益」记为 `f[i][2]`。

> 这里的「处于冷冻期」指的是在第 i 天结束之后的状态。也就是说：如果第 i 天结束之后处于冷冻期，那么第 i+1 天无法买入股票。

```
对于 f[i][0]，我们目前持有的这一支股票可以是在第 i-1 天就已经持有的，对应的状态为 f[i-1][0]；或者是第 i 天买入的，那么第 i-1 天就不能持有股票并且不处于冷冻期中，对应的状态为 f[i-1][2] 加上买入股票的负收益 prices[i]。因此状态转移方程为：

f[i][0]=max(f[i−1][0],f[i−1][2]−prices[i])

对于 f[i][1]，我们在第 i 天结束之后处于冷冻期的原因是在当天卖出了股票，那么说明在第 i-1 天时我们必须持有一支股票，对应的状态为 f[i-1][0] 加上卖出股票的正收益 prices[i]。因此状态转移方程为：

f[i][1]=f[i−1][0]+prices[i]

对于 f[i][2]，我们在第 i 天结束之后不持有任何股票并且不处于冷冻期，说明当天没有进行任何操作，即第 i-1 天时不持有任何股票：如果处于冷冻期，对应的状态为 f[i-1][1]；如果不处于冷冻期，对应的状态为 f[i-1][2]。因此状态转移方程为：

f[i][2]=max(f[i−1][1],f[i−1][2])

这样我们就得到了所有的状态转移方程。如果一共有 n 天，那么最终的答案即为：

max(f[n−1][0],f[n−1][1],f[n−1][2])

注意到如果在最后一天（第 n-1 天）结束之后，手上仍然持有股票，那么显然是没有任何意义的。因此更加精确地，最终的答案实际上是 f[n-1][1] 和 f[n-1][2] 中的较大值，即：

max(f[n−1][1],f[n−1][2])
```

我们可以将第 0 天的情况作为动态规划中的边界条件：

- `f[0][0]=−prices[0]`
- `f[0][1]=0`
- `f[0][2]=0`

在第 0 天时，如果持有股票，那么只能是在第 0 天买入的，对应负收益 −prices[0]；如果不持有股票，那么收益为零。注意到第 0 天实际上是不存在处于冷冻期的情况的，但我们仍然可以将对应的状态 `f[0][1]` 置为零

这样我们就可以从第 1 天开始，根据上面的状态转移方程进行进行动态规划，直到计算出第 n-1 天的结果。

```java
class Solution {
    public int maxProfit(int[] prices) {
        if (prices.length == 0) {
            return 0;
        }
        int n = prices.length;
        // f[i][0]: 手上持有股票的最大收益
        // f[i][1]: 手上不持有股票，并且处于冷冻期中的累计最大收益
        // f[i][2]: 手上不持有股票，并且不在冷冻期中的累计最大收益
        int[][] f = new int[n][3];
        f[0][0] = -prices[0];
        for (int i = 1; i < n; ++i) {
            f[i][0] = Math.max(f[i - 1][0], f[i - 1][2] - prices[i]);
            f[i][1] = f[i - 1][0] + prices[i];
            f[i][2] = Math.max(f[i - 1][1], f[i - 1][2]);
        }
        return Math.max(f[n - 1][1], f[n - 1][2]);
    }
}
```

- 时间复杂度：O(n)，其中 n 为数组的长度。
- 空间复杂度：O(n)，我们需要 3n 的空间存储动态规划中的所有状态，对应的空间复杂度为 O(n)。

空间优化

注意到上面的状态转移方程中，`f[i][..]` 只与 `f[i-1][..]` 有关，而与 `f[i-2][..] `及之前的所有状态都无关，因此我们不必存储这些无关的状态。也就是说，我们只需要将 `f[i-1][0]，f[i-1][1]，f[i-1][2] `存放在三个变量中，通过它们计算出 `f[i][0]，f[i][1]，f[i][2] `并存回对应的变量，以便于第 `i+1` 天的状态转移即可。

```java
class Solution {
    public int maxProfit(int[] prices) {
        if (prices.length == 0) {
            return 0;
        }

        int n = prices.length;
        int f0 = -prices[0];
        int f1 = 0;
        int f2 = 0;
        for (int i = 1; i < n; ++i) {
            int newf0 = Math.max(f0, f2 - prices[i]);
            int newf1 = f0 + prices[i];
            int newf2 = Math.max(f1, f2);
            f0 = newf0;
            f1 = newf1;
            f2 = newf2;
        }

        return Math.max(f1, f2);
    }
}
```

- 时间复杂度：O(n)，其中 n 为数组的长度。
- 空间复杂度：O(1)。

### [030] 颜色分类

给定一个包含红色、白色和蓝色，一共 n 个元素的数组，原地对它们进行排序，使得相同颜色的元素相邻，并按照红色、白色、蓝色顺序排列。

此题中，我们使用整数 0、 1 和 2 分别表示红色、白色和蓝色。

进阶：

你可以不使用代码库中的排序函数来解决这道题吗？
你能想出一个仅使用常数空间的一趟扫描算法吗？

```java
示例 1：

输入：nums = [2,0,2,1,1,0]
输出：[0,0,1,1,2,2]
示例 2：

输入：nums = [2,0,1]
输出：[0,1,2]
示例 3：

输入：nums = [0]
输出：[0]
示例 4：

输入：nums = [1]
输出：[1]


提示：

n == nums.length
1 <= n <= 300
nums[i] 为 0、1 或 2
```

方法一：单指针两次遍历

我们可以考虑对数组进行两次遍历。在第一次遍历中，我们将数组中所有的 0 交换到数组的头部。在第二次遍历中，我们将数组中所有的 1 交换到头部的 0 之后。此时，所有的 2 都出现在数组的尾部，这样我们就完成了排序。

```java
class Solution {
    public void sortColors(int[] nums) {
        int n = nums.length;
        int ptr = 0;
        //从左向右遍历整个数组
        //如果找到了 0，那么就需要将 0 与「头部」位置的元素进行交换
        //并将「头部」向后扩充一个位置
        for (int i = 0; i < n; ++i) {
            if (nums[i] == 0) {
                int temp = nums[i];
                nums[i] = nums[ptr];
                nums[ptr] = temp;
                ++ptr;
            }
        }
        //从「头部」开始，从左向右遍历整个数组
        //如果找到了 1，那么就需要将 1 与「头部」位置的元素进行交换
        //并将「头部」向后扩充一个位置
        for (int i = ptr; i < n; ++i) {
            if (nums[i] == 1) {
                int temp = nums[i];
                nums[i] = nums[ptr];
                nums[ptr] = temp;
                ++ptr;
            }
        }
    }
}
```

- 时间复杂度：O(n)，其中 n 是数组 nums 的长度。
- 空间复杂度：O(1)。

方法二：双指针一次遍历

可以额外使用一个指针，即使用两个指针分别用来交换 0 和 1。

```java
class Solution {
    public void sortColors(int[] nums) {
        int n = nums.length;
        //指针 p0来交换0，p1来交换1，初始值都为0。
        int p0 = 0, p1 = 0;
        for (int i = 0; i < n; ++i) {
            //找到了1，那么将其与nums[p1] 进行交换，并将 p1向后移动一个位置
            if (nums[i] == 1) {
                int temp = nums[i];
                nums[i] = nums[p1];
                nums[p1] = temp;
                ++p1;
            } else if (nums[i] == 0) {
                //找到了0,将其与 nums[p0] 进行交换
                int temp = nums[i];
                nums[i] = nums[p0];
                nums[p0] = temp;
                //当 p0 < p1 时
                //再将nums[i] 与nums[p1] 进行交换
                if (p0 < p1) {
                    temp = nums[i];
                    nums[i] = nums[p1];
                    nums[p1] = temp;
                }
                //p0和p1均向后移动一个位置
                ++p0;
                ++p1;
            }
        }
    }
}
```

- 时间复杂度：O(n)，其中 n 是数组 nums 的长度。
- 空间复杂度：O(1)。

### [031] 除法求值※

给你一个变量对数组 `equations` 和一个实数值数组 `values` 作为已知条件，其中 `equations[i] = [Ai, Bi]` 和 `values[i]`共同表示等式 `Ai / Bi = values[i]` 。每个 `Ai `或 `Bi` 是一个表示单个变量的字符串。

另有一些以数组 `queries` 表示的问题，其中 `queries[j] = [Cj, Dj]`表示第 `j` 个问题，请你根据已知条件找出 `Cj / Dj = ? `的结果作为答案。

返回 所有问题的答案 。如果存在某个无法确定的答案，则用 `-1.0` 替代这个答案。如果问题中出现了给定的已知条件中没有出现的字符串，也需要用 `-1.0` 替代这个答案。

注意：输入总是有效的。你可以假设除法运算中不会出现除数为 0 的情况，且不存在任何矛盾的结果。

 示例 1：

```
输入：equations = [["a","b"],["b","c"]], values = [2.0,3.0], queries = [["a","c"],["b","a"],["a","e"],["a","a"],["x","x"]]
输出：[6.00000,0.50000,-1.00000,1.00000,-1.00000]
解释：
条件：a / b = 2.0, b / c = 3.0
问题：a / c = ?, b / a = ?, a / e = ?, a / a = ?, x / x = ?
结果：[6.0, 0.5, -1.0, 1.0, -1.0 ]
```

示例 2：

```
输入：equations = [["a","b"],["b","c"],["bc","cd"]], values = [1.5,2.5,5.0], queries = [["a","c"],["c","b"],["bc","cd"],["cd","bc"]]
输出：[3.75000,0.40000,5.00000,0.20000]
```

示例 3：

```
输入：equations = [["a","b"]], values = [0.5], queries = [["a","b"],["b","a"],["a","c"],["x","y"]]
输出：[0.50000,2.00000,-1.00000,-1.00000]
```


提示：

- 1 <= equations.length <= 20
- equations[i].length == 2
- 1 <= Ai.length, Bi.length <= 5
- values.length == equations.length
- 0.0 < values[i] <= 20.0
- 1 <= queries.length <= 20
- queries[i].length == 2
- 1 <= Cj.length, Dj.length <= 5
- Ai, Bi, Cj, Dj 由小写英文字母与数字组成

方法一：并查集

[题解](https://leetcode-cn.com/problems/evaluate-division/solution/399-chu-fa-qiu-zhi-nan-du-zhong-deng-286-w45d/)

```java
class Solution {
    public double[] calcEquation(List<List<String>> equations, double[] values, List<List<String>> queries) {
     int equationsSize = equations.size();

        UnionFind unionFind = new UnionFind(2 * equationsSize);
        // 第 1 步：预处理，将变量的值与 id 进行映射，使得并查集的底层使用数组实现，方便编码
        Map<String, Integer> hashMap = new HashMap<>(2 * equationsSize);
        int id = 0;
        for (int i = 0; i < equationsSize; i++) {
            List<String> equation = equations.get(i);
            String var1 = equation.get(0);
            String var2 = equation.get(1);

            if (!hashMap.containsKey(var1)) {
                hashMap.put(var1, id);
                id++;
            }
            if (!hashMap.containsKey(var2)) {
                hashMap.put(var2, id);
                id++;
            }
            unionFind.union(hashMap.get(var1), hashMap.get(var2), values[i]);
        }

        // 第 2 步：做查询
        int queriesSize = queries.size();
        double[] res = new double[queriesSize];
        for (int i = 0; i < queriesSize; i++) {
            String var1 = queries.get(i).get(0);
            String var2 = queries.get(i).get(1);

            Integer id1 = hashMap.get(var1);
            Integer id2 = hashMap.get(var2);

            if (id1 == null || id2 == null) {
                res[i] = -1.0d;
            } else {
                res[i] = unionFind.isConnected(id1, id2);
            }
        }
        return res;
    }

    private class UnionFind {
        private int[] parent;

        /**
         * 指向的父结点的权值
         */
        private double[] weight;

        public UnionFind(int n) {
            this.parent = new int[n];
            this.weight = new double[n];
            for (int i = 0; i < n; i++) {
                parent[i] = i;
                weight[i] = 1.0d;
            }
        }

        public void union(int x, int y, double value) {
            int rootX = find(x);
            int rootY = find(y);
            if (rootX == rootY) {
                return;
            }

            parent[rootX] = rootY;
            // 关系式的推导请见「参考代码」下方的示意图
            weight[rootX] = weight[y] * value / weight[x];
        }

        /**
         * 路径压缩
         *
         * @param x
         * @return 根结点的 id
         */
        public int find(int x) {
            if (x != parent[x]) {
                int origin = parent[x];
                parent[x] = find(parent[x]);
                weight[x] *= weight[origin];
            }
            return parent[x];
        }

        public double isConnected(int x, int y) {
            int rootX = find(x);
            int rootY = find(y);
            if (rootX == rootY) {
                return weight[x] / weight[y];
            } else {
                return -1.0d;
            }
        }
    }
}
```

### [032] 路径总和 III

给定一个二叉树，它的每个结点都存放着一个整数值。

找出路径和等于给定数值的路径总数。

路径不需要从根节点开始，也不需要在叶子节点结束，但是路径方向必须是向下的（只能从父节点到子节点）。

二叉树不超过1000个节点，且节点数值范围是 [-1000000,1000000] 的整数。

示例：

    root = [10,5,-3,3,2,null,11,3,-2,null,1], sum = 8      
          10
         /  \
        5   -3
       / \    \
      3   2   11
     / \   \
    3  -2   1
    
    返回 3。和等于 8 的路径有:
    
    1.  5 -> 3
    2.  5 -> 2 -> 1
    3.  -3 -> 11
方法一：前缀和，递归，回溯

前缀和：到达当前元素的路径上，之前所有元素的和。

如果两个数的**前缀总和**是相同的，那么这些**节点之间的元素总和为零**。进一步扩展相同的想法，如果前缀总和`currSum`，在节点A和节点B处相差`target`，则位于节点A和节点B之间的元素之和是`target`。

因为本题中的路径是一棵树，从根往任一节点的路径上(不走回头路)，**有且仅有一条路径**，**不存在环**。(如果存在环，前缀和就不能用了，需要改造算法)

抵达当前节点(即B节点)后，将前缀和累加，然后查找在前缀和上，有没有**前缀和`currSum-target`的节点**(即A节点)，存在即表示从A到B有一条路径之和满足条件的情况。结果加上满足前缀和`currSum-target`的节点的数量。然后递归进入左右子树。

左右子树遍历完成之后，回到当前层，需要把当前节点添加的前缀和去除。避免回溯之后影响上一层。

核心代码：

```java
// 当前路径上的和
currSum += node.val;
// currSum-target相当于找路径的起点，起点的sum+target=currSum，当前点到起点的距离就是target
res += prefixSumCount.getOrDefault(currSum - target, 0);
// 更新路径上当前节点前缀和的个数
prefixSumCount.put(currSum, prefixSumCount.getOrDefault(currSum, 0) + 1);
```

完整代码：

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
    public int pathSum(TreeNode root, int sum) {
        // key是前缀和, value是大小为key的前缀和出现的次数
        Map<Integer, Integer> prefixSumCount = new HashMap<>();
        // 前缀和为0的一条路径
        prefixSumCount.put(0, 1);
        // 前缀和的递归回溯思路
        return recursionPathSum(root, prefixSumCount, sum, 0);
    }

    /**
     * 前缀和的递归回溯思路
     * 从当前节点反推到根节点(反推比较好理解，正向其实也只有一条)，有且仅有一条路径，因为这是一棵树
     * 如果此前有和为currSum-target,而当前的和又为currSum,两者的差就肯定为target了
     * 所以前缀和对于当前路径来说是唯一的，当前记录的前缀和，在回溯结束，回到本层时去除，保证其不影响其他分支的结果
     * @param node 树节点
     * @param prefixSumCount 前缀和Map
     * @param target 目标值
     * @param currSum 当前路径和
     * @return 满足题意的解
     */
    private int recursionPathSum(TreeNode node, Map<Integer, Integer> prefixSumCount, int target, int currSum) {
        // 1.递归终止条件
        if (node == null) {
            return 0;
        }
        // 2.本层要做的事情
        int res = 0;
        // 当前路径上的和
        currSum += node.val;

        //---核心代码
        // 看看root到当前节点这条路上是否存在节点前缀和加target为currSum的路径
        // 当前节点->root节点反推，有且仅有一条路径，如果此前有和为currSum-target,而当前的和又为currSum,两者的差就肯定为target了
        // currSum-target相当于找路径的起点，起点的sum+target=currSum，当前点到起点的距离就是target
        res += prefixSumCount.getOrDefault(currSum - target, 0);
        // 更新路径上当前节点前缀和的个数
        prefixSumCount.put(currSum, prefixSumCount.getOrDefault(currSum, 0) + 1);
        //---核心代码

        // 3.进入下一层
        res += recursionPathSum(node.left, prefixSumCount, target, currSum);
        res += recursionPathSum(node.right, prefixSumCount, target, currSum);

        // 4.回到本层，恢复状态，去除当前节点的前缀和数量
        prefixSumCount.put(currSum, prefixSumCount.get(currSum) - 1);
        return res;
    }
}
```

时间复杂度：O(N)，每个节点只遍历一次

空间复杂度：O(N)，开辟了一个hashMap

方法二：暴力

需要去求三部分即可：

- 以当前节点作为头结点的路径数量
- 当前节点的左子树中满足条件的路径数量
- 当前节点的右子树中满足条件的路径数量

将这三部分之和作为最后结果即可。

最后的问题是：如何去求以当前节点作为头结点的路径的数量？这里依旧是按照树的遍历方式模板，每到一个节点让`sum-root.val`，并判断sum是否为0，如果为零的话，则找到满足条件的一条路径。

```java
class Solution {
    public int pathSum(TreeNode root, int sum) {
        if(root == null){
            return 0;
        }
        int result = countPath(root,sum);
        int a = pathSum(root.left,sum);
        int b = pathSum(root.right,sum);
        return result+a+b;

    }
    public int countPath(TreeNode root,int sum){
        if(root == null){
            return 0;
        }
        sum = sum - root.val;
        int result = sum == 0 ? 1:0;
        return result + countPath(root.left,sum) + countPath(root.right,sum);
    }
}
```

时间复杂度：O(N^2)，每个节点只遍历一次

空间复杂度：O(H)，H为树的高度；

### [033] 电话号码的字母组合

给定一个仅包含数字 2-9 的字符串，返回所有它能表示的字母组合。

给出数字到字母的映射如下（与电话按键相同）。注意 1 不对应任何字母。

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/original_images/17_telephone_keypad.png)

示例:

```
输入："23"
输出：["ad", "ae", "af", "bd", "be", "bf", "cd", "ce", "cf"].
```

说明:

- 尽管上面的答案是按字典序排列的，但是你可以任意选择答案输出的顺序。

方法一·：回溯解法



```java
class Solution {
   //一个映射表，第二个位置是"abc“,第三个位置是"def"。。。
    //这里也可以用map，用数组可以更节省点内存
    String[] letter_map = {" ","*","abc","def","ghi","jkl","mno","pqrs","tuv","wxyz"};
    public List<String> letterCombinations(String digits) {
        //注意边界条件
        if(digits==null || digits.length()==0) {
            return new ArrayList<>();
        }
        iterStr(digits, new StringBuilder(), 0);
        return res;
    }
    //最终输出结果的list
    List<String> res = new ArrayList<>();

    //递归函数
    private void iterStr(String str, StringBuilder letter, int index) {
        //递归的终止条件
        //用index记录每次遍历到字符串的位置
        if(index == str.length()) {
            res.add(letter.toString());
            return;
        }
        //获取index位置的字符，假设输入的字符是"234"
        //第一次递归时index为0所以c=2，第二次index为1所以c=3，第三次c=4
        //subString每次都会生成新的字符串，而index则是取当前的一个字符，所以效率更高一点
        char c = str.charAt(index);
        //map_string的下表是从0开始一直到9， c-'0'就可以取到相对的数组下标位置
        //比如c=2时候，2-'0'，获取下标为2,letter_map[2]就是"abc"
        int pos = c - '0';
        String map_string = letter_map[pos];
        //遍历字符串，比如第一次得到的是2，页就是遍历"abc"
        for(int i=0;i<map_string.length();i++) {
            //调用下一层递归
            letter.append(map_string.charAt(i));
            iterStr(str, letter, index+1);
            //回溯
            letter.deleteCharAt(letter.length()-1);
        }
    }
}
```

时间复杂度：O(3^n)这个级别

空间复杂度：O(n)

方法二：利用队列求解

我们可以利用队列的先进先出特点，再配合循环完成题目要求。

动态演示如下：

![队列-动态图.gif](https://pic.leetcode-cn.com/6953e7a27bff1242c37f88c9b66b524975655605d053a9f6ac6a74376582b4c5-%E9%98%9F%E5%88%97-%E5%8A%A8%E6%80%81%E5%9B%BE.gif)

```java
class Solution {
	public List<String> letterCombinations(String digits) {
		if(digits==null || digits.length()==0) {
			return new ArrayList<String>();
		}
		//一个映射表，第二个位置是"abc“,第三个位置是"def"。。。
		//这里也可以用map，用数组可以更节省点内存
		String[] letter_map = {
			" ","*","abc","def","ghi","jkl","mno","pqrs","tuv","wxyz"
		};
		List<String> res = new ArrayList<>();
		//先往队列中加入一个空字符
		res.add("");
		for(int i=0;i<digits.length();i++) {
			//由当前遍历到的字符，取字典表中查找对应的字符串
			String letters = letter_map[digits.charAt(i)-'0'];
			int size = res.size();
			//计算出队列长度后，将队列中的每个元素挨个拿出来
			for(int j=0;j<size;j++) {
				//每次都从队列中拿出第一个元素
				String tmp = res.remove(0);
				//然后跟"def"这样的字符串拼接，并再次放到队列中
				for(int k=0;k<letters.length();k++) {
					res.add(tmp+letters.charAt(k));
				}
			}
		}
		return res;
	}
}
```

时间复杂度：O(3^M×4^N)。M 是对应三个字母的数字个数，N 是对应四个字母的数字个数。
空间复杂度：O(3^M×4^N)。一共要生成 3^M×4^N个结果。

### [034] 任务调度器

给你一个用字符数组 tasks 表示的 CPU 需要执行的任务列表。其中每个字母表示一种不同种类的任务。任务可以以任意顺序执行，并且每个任务都可以在 1 个单位时间内执行完。在任何一个单位时间，CPU 可以完成一个任务，或者处于待命状态。

然而，两个 相同种类 的任务之间必须有长度为整数 n 的冷却时间，因此至少有连续 n 个单位时间内 CPU 在执行不同的任务，或者在待命状态。

你需要计算完成所有任务所需要的 最短时间 。

 

示例 1：

```
输入：tasks = ["A","A","A","B","B","B"], n = 2
输出：8
解释：A -> B -> (待命) -> A -> B -> (待命) -> A -> B
     在本示例中，两个相同类型任务之间必须间隔长度为 n = 2 的冷却时间，而执行一个任务只需要一个单位时间，所以中间出现了（待命）状态。 
```

示例 2：

```
输入：tasks = ["A","A","A","B","B","B"], n = 0
输出：6
解释：在这种情况下，任何大小为 6 的排列都可以满足要求，因为 n = 0
["A","A","A","B","B","B"]
["A","B","A","B","A","B"]
["B","B","B","A","A","A"]
...
诸如此类
```

示例 3：

```
输入：tasks = ["A","A","A","A","A","A","B","C","D","E","F","G"], n = 2
输出：16
解释：一种可能的解决方案是：
     A -> B -> C -> A -> D -> E -> A -> F -> G -> A -> (待命) -> (待命) -> A -> (待命) -> (待命) -> A
```


提示：

- 1 <= task.length <= 10^4
- tasks[i] 是大写英文字母
- n 的取值范围为 [0, 100]

方法一：贪心

贪心策略为：先安排出现次数最多的任务，让这个任务两次执行的时间间隔正好为n。再在这个时间间隔内填充其他的任务。

例如：tasks = ["A","A","A","B","B","B"], n = 2

我们先安排出现次数最多的任务"A",并且让两次执行"A"的时间间隔为2。在这个时间间隔内，我们用其他任务类型去填充，又因为其他任务类型只有"B"一个，不够填充2的时间间隔，因此额外需要一个冷却时间间隔。具体安排如下图所示：

![621.png](https://pic.leetcode-cn.com/1607137838-cisnuO-621.png)

其中，maxTimes为出现次数最多的那个任务出现的次数。maxCount为一共有多少个任务和出现最多的那个任务出现次数一样。

图中一共占用的方格即为完成所有任务需要的时间，即：

```
(maxTimes−1)∗(n + 1) + maxCount
```

此外，如果任务种类很多，在安排时无需冷却时间，只需要在一个任务的两次出现间填充其他任务，然后从左到右从上到下依次执行即可，由于每一个任务占用一个时间单位，我们又正正好好地使用了tasks中的所有任务，而且我们只使用tasks中的任务来占用方格（没用冷却时间）。因此这种情况下，所需要的时间即为tasks的长度。

综上，只需把公式计算的结果和tasks的长度取最大即为最终结果。

```java
class Solution {
    public int leastInterval(char[] tasks, int n) {
        //题目限定 tasks[i] 是大写英文字母
        int[] buckets = new int[26];
        //统级出现次数，保存到数组
        for(int i = 0; i < tasks.length; i++){
            buckets[tasks[i] - 'A']++;
        }
        //排序
        Arrays.sort(buckets);
        //找出出现次数最多的
        int maxTimes = buckets[25];
        //一共有多少个任务和出现最多的那个任务出现次数一样
        int maxCount = 1;
        for(int i = 25; i >= 1; i--){
            if(buckets[i] == buckets[i - 1])
                maxCount++;
            else
                break;
        }
        //完成所有任务需要的时间,见上面分析
        int res = (maxTimes - 1) * (n + 1) + maxCount;
        return Math.max(res, tasks.length);
    }
}
```

时间复杂度：O(∣task∣+∣Σ∣)，其中∣Σ∣ 是数组 task 中出现任务的种类，在本题中任务用大写字母表示，因此 ∣Σ∣ 不会超过 26。

空间复杂度：O(∣Σ∣)。

### [035] 课程表

你这个学期必须选修 numCourse 门课程，记为 0 到 numCourse-1 。

在选修某些课程之前需要一些先修课程。 例如，想要学习课程 0 ，你需要先完成课程 1 ，我们用一个匹配来表示他们：[0,1]

给定课程总量以及它们的先决条件，请你判断是否可能完成所有课程的学习？

示例 1:

```
输入: 2, [[1,0]] 
输出: true
解释: 总共有 2 门课程。学习课程 1 之前，你需要完成课程 0。所以这是可能的。
```

示例 2:

```
输入: 2, [[1,0],[0,1]]
输出: false
解释: 总共有 2 门课程。学习课程 1 之前，你需要先完成课程 0；并且学习课程 0 之前，你还应先完成课程 1。这是不可能的。
```


提示：

- 输入的先决条件是由 边缘列表 表示的图形，而不是 邻接矩阵 。详情请参见图的表示法。
- 你可以假定输入的先决条件中没有重复的边。
- 1 <= numCourses <= 10^5

解题思路：

- 本题可约化为： 课程安排图是否是 **有向无环图**(DAG)。即课程间规定了前置条件，但不能构成任何环路，否则课程前置条件将不成立。
- 思路是通过 **拓扑排序** 判断此课程安排图是否是 **有向无环图**(DAG) 。 拓扑排序原理： 对 DAG 的顶点进行排序，使得对每一条有向边 `(u, v)`，均有 u（在排序记录中）比 v 先出现。亦可理解为对某点 v 而言，只有当 v 的所有源点均出现了，v 才能出现。
- 通过课程前置条件列表 `prerequisites` 可以得到课程安排图的 **邻接表** `adjacency`，以降低算法时间复杂度，以下两种方法都会用到邻接表。

方法一：入度表（广度优先遍历）

- 统计课程安排图中每个节点的入度，生成 入度表 `indegrees`。
- 借助一个队列 queue，将所有入度为 0 的节点入队。
- 当 queue 非空时，依次将队首节点出队，在课程安排图中删除此节点 pre：
  - 并不是真正从邻接表中删除此节点 pre，而是将此节点对应所有邻接节点 cur 的入度 −1，即 `indegrees[cur] -= 1`。
  - 当入度 −1后邻接节点 cur 的入度为 0，说明 cur 所有的前驱节点已经被 “删除”，此时将 cur 入队。
- 在每次 pre 出队时，执行 `numCourses--`；
  - 若整个课程安排图是有向无环图（即可以安排），则所有节点一定都入队并出队过，即完成拓扑排序。换个角度说，若课程安排图中存在环，一定有节点的入度始终不为 0。
  - 因此，拓扑排序出队次数等于课程个数，返回 `numCourses == 0` 判断课程是否可以成功安排。

```java
class Solution {
    public boolean canFinish(int numCourses, int[][] prerequisites) {
        int[] indegrees = new int[numCourses];
        List<List<Integer>> adjacency = new ArrayList<>();
        Queue<Integer> queue = new LinkedList<>();
        for(int i = 0; i < numCourses; i++)
            adjacency.add(new ArrayList<>());
        // Get the indegree and adjacency of every course.
        for(int[] cp : prerequisites) {
            indegrees[cp[0]]++;
            adjacency.get(cp[1]).add(cp[0]);
        }
        // Get all the courses with the indegree of 0.
        for(int i = 0; i < numCourses; i++)
            if(indegrees[i] == 0) queue.add(i);
        // BFS TopSort.
        while(!queue.isEmpty()) {
            int pre = queue.poll();
            numCourses--;
            for(int cur : adjacency.get(pre))
                if(--indegrees[cur] == 0) queue.add(cur);
        }
        return numCourses == 0;
    }
}
```

时间复杂度 O(N + M)： 遍历一个图需要访问所有节点和所有临边，N 和 M 分别为节点数量和临边数量；

空间复杂度 O(N + M)： 为建立邻接表所需额外空间，adjacency 长度为 N ，并存储 M 条临边的数据。

方法二：深度优先遍历

原理是通过 DFS 判断图中是否有环。

- 借助一个标志列表 flags，用于判断每个节点 i （课程）的状态：
  - 未被 DFS 访问：`i == 0`；
  - 已被其他节点启动的 DFS 访问：`i == -1`；
  - 已被当前节点启动的 DFS 访问：`i == 1`。
- 对 numCourses 个节点依次执行 DFS，判断每个节点起步 DFS 是否存在环，若存在环直接返回 FalseFalse。DFS 流程；
  - 终止条件：
    - 当 `flag[i] == -1`，说明当前访问节点已被其他节点启动的 DFS 访问，无需再重复搜索，直接返回 True。
    - 当 `flag[i] == 1`，说明在本轮 DFS 搜索中节点 i 被第 2 次访问，即 **课程安排图有环** ，直接返回 False。
  - 将当前访问节点 i 对应 `flag[i]` 置 1，即标记其被本轮 DFS 访问过；
  - 递归访问当前节点 i 的所有邻接节点 j，当发现环直接返回 False；
  - 当前节点所有邻接节点已被遍历，并没有发现环，则将当前节点 flag 置为 −1 并返回 True。
- 若整个图 DFS 结束并未发现环，返回 True。

```java
class Solution {
    public boolean canFinish(int numCourses, int[][] prerequisites) {
        List<List<Integer>> adjacency = new ArrayList<>();
        for(int i = 0; i < numCourses; i++)
            adjacency.add(new ArrayList<>());
        int[] flags = new int[numCourses];
        for(int[] cp : prerequisites)
            adjacency.get(cp[1]).add(cp[0]);
        for(int i = 0; i < numCourses; i++)
            if(!dfs(adjacency, flags, i)) return false;
        return true;
    }
    private boolean dfs(List<List<Integer>> adjacency, int[] flags, int i) {
        if(flags[i] == 1) return false;
        if(flags[i] == -1) return true;
        flags[i] = 1;
        for(Integer j : adjacency.get(i))
            if(!dfs(adjacency, flags, j)) return false;
        flags[i] = -1;
        return true;
    }
}
```

时间复杂度 O(N + M)： 遍历一个图需要访问所有节点和所有临边，N 和 M 分别为节点数量和临边数量；
空间复杂度 O(N + M)： 为建立邻接表所需额外空间，adjacency 长度为 N ，并存储 M 条临边的数据。

