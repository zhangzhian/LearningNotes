# Leetcode数据结构与算法

### [0145] 斐波那契数

斐波那契数，通常用 F(n) 表示，形成的序列称为**斐波那契数列**。该数列由 0 和 1 开始，后面的每一项数字都是前面两项数字的和。也就是：

```
F(0) = 0,   F(1) = 1
F(N) = F(N - 1) + F(N - 2), 其中 N > 1.
```

给定 N，计算 F(N)。

 示例 1：

```
输入：2
输出：1
解释：F(2) = F(1) + F(0) = 1 + 0 = 1.
```

示例 2：

```
输入：3
输出：2
解释：F(3) = F(2) + F(1) = 1 + 1 = 2.
```

示例 3：

```
输入：4
输出：3
解释：F(4) = F(3) + F(2) = 2 + 1 = 3.
```


提示：

`0 ≤ N ≤ 30`

方法一：递归

```java
public class Solution {
    public int fib(int N) {
        if (N <= 1) {
            return N;
        }
        return fib(N-1) + fib(N-2);
    }
}
```

时间复杂度：O(2^N)。这是计算斐波那契数最慢的方法。因为它需要指数的时间。
空间复杂度：O(N)，在堆栈中我们需要与 N 成正比的空间大小。该堆栈跟踪 fib(N) 的函数调用，随着堆栈的不断增长如果没有足够的内存则会导致 StackOverflowError。

方法二：记忆化自底向上的方法

自底向上通过迭代计算斐波那契数的子问题并存储已计算的值，通过已计算的值进行计算。减少递归带来的重复计算。

```java
class Solution {
    public int fib(int N) {
        if (N <= 1) {
            return N;
        }
        return memoize(N);
    }

    public int memoize(int N) {
      int[] cache = new int[N + 1];
      cache[1] = 1;

      for (int i = 2; i <= N; i++) {
          cache[i] = cache[i-1] + cache[i-2];
      }
      return cache[N];
    }
}
```

- 时间复杂度：O(N)。
- 空间复杂度：O(N)，使用了空间大小为 `N` 的数组。

方法三：记忆化自顶向下的方法

我们先计算存储子问题的答案，然后利用子问题的答案计算当前斐波那契数的答案。我们将递归计算，但是通过记忆化不重复计算已计算的值。

```java
class Solution {
    private Integer[] cache = new Integer[31];

    public int fib(int N) {
        if (N <= 1) {
            return N;
        }
        cache[0] = 0;
        cache[1] = 1;
        return memoize(N);
    }

    public int memoize(int N) {
      if (cache[N] != null) {
          return cache[N];
      }
      cache[N] = memoize(N-1) + memoize(N-2);
      return memoize(N);
    }
}
```

- 时间复杂度：O(N)。
- 空间复杂度：O(N)，内存中使用的堆栈大小。

方法四：自底向上进行迭代

```java
class Solution {
    public int fib(int N) {
        if (N <= 1) {
            return N;
        }
        if (N == 2) {
            return 1;
        }

        int current = 0;
        int prev1 = 1;
        int prev2 = 1;

        for (int i = 3; i <= N; i++) {
            current = prev1 + prev2;
            prev2 = prev1;
            prev1 = current;
        }
        return current;
    }
}
```

- 时间复杂度：O(N)。
- 空间复杂度：O(1)，仅仅使用了 `current`，`prev1`，`prev2`。

方法五：矩阵求幂

斐波那契数列矩阵方程：

![斐波那契数列矩阵方程](https://pic.leetcode-cn.com/09fd107ad5ce57b564aaae069bacd1b04fbb5033197d99305e3a45764bf07fee.png)

算法：

- 若 N 小于等一 1，则返回 N。
- 使用递归函数 matrixPower 计算给定矩阵 A 的幂。幂为 N-1，其中 N 是第 N 个 斐波那契
- matrixPower 函数将对 N/2 个斐波那契数进行操作。
- 在 matrixPower 中，调用 multiply 函数将两个矩阵相乘。
- 完成计算后，返回 `A[0][0]` 得到第 N 个斐波那契数。

```java
class Solution {
    int fib(int N) {
        if (N <= 1) {
          return N;
        }
        int[][] A = new int[][]{{1, 1}, {1, 0}};
        matrixPower(A, N-1);

        return A[0][0];
    }

    void matrixPower(int[][] A, int N) {
        if (N <= 1) {
          return;
        }
        matrixPower(A, N/2);
        multiply(A, A);

        int[][] B = new int[][]{{1, 1}, {1, 0}};
        if (N%2 != 0) {
            multiply(A, B);
        }
    }

    void multiply(int[][] A, int[][] B) {
        int x = A[0][0] * B[0][0] + A[0][1] * B[1][0];
        int y = A[0][0] * B[0][1] + A[0][1] * B[1][1];
        int z = A[1][0] * B[0][0] + A[1][1] * B[1][0];
        int w = A[1][0] * B[0][1] + A[1][1] * B[1][1];

        A[0][0] = x;
        A[0][1] = y;
        A[1][0] = z;
        A[1][1] = w;
    }
}
```

- 时间复杂度：O(logN)。
- 空间复杂度：O(logN)，`matrixPower` 函数递归时堆栈使用的空间。

方法六：公式法

- 使用黄金分割比：
  $$
  \varphi = \frac{1 + \sqrt{5}}{2} \approx 1.6180339887....φ= 
  2
  1+ 
  5
  ​	
   
  ​	
   ≈1.6180339887....
  $$
  Binet公式 ：

![在这里插入图片描述](https://pic.leetcode-cn.com/d5e05b3f46910d4bf79d3b235e5a3d7803b63bce092c6c20a53c16d228e33113.png)

- 这里有一个[链接](http://demonstrations.wolfram.com/GeneralizedFibonacciSequenceAndTheGoldenRatio/)，可以进一步了解斐波那契序列和黄金分割比之间的关系。
- 我们可以只能用常数时间和常数空间得到斐波那契数。

算法：

使用黄金分割率计算第 `N` 个斐波那契数。

```java
class Solution {
    public int fib(int N) {
        double goldenRatio = (1 + Math.sqrt(5)) / 2;
        return (int)Math.round(Math.pow(goldenRatio, N)/ Math.sqrt(5));
    }
}	
```

时间复杂度：O(1)。常数的时间复杂度，因为我们是基于 Binet 公式进行计算，没有使用循环或递归。
空间复杂度：O(1)，存储黄金分割率所使用的空间。

### [0146] 二叉树的所有路径

 给定一个二叉树，返回所有从根节点到叶子节点的路径。

说明: 叶子节点是指没有子节点的节点。

示例:

```java
输入:

   1
 /   \
2     3
 \
  5

输出: ["1->2->5", "1->3"]

解释: 所有根节点到叶子节点的路径为: 1->2->5, 1->3。
```

方法一：深度优先搜索

最直观的方法是使用深度优先搜索。在深度优先搜索遍历二叉树时，我们需要考虑当前的节点以及它的孩子节点。

- 如果当前节点不是叶子节点，则在当前的路径末尾添加该节点，并继续递归遍历该节点的每一个孩子节点。
- 如果当前节点是叶子节点，则在当前路径末尾添加该节点后我们就得到了一条从根节点到叶子节点的路径，将该路径加入到答案即可。

如此，当遍历完整棵二叉树以后我们就得到了所有从根节点到叶子节点的路径。当然，深度优先搜索也可以使用非递归的方式实现。

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
    public List<String> binaryTreePaths(TreeNode root) {
        List<String> paths = new ArrayList<String>();
        constructPaths(root, "", paths);
        return paths;
    }

    public void constructPaths(TreeNode root, String path, List<String> paths) {
        if (root != null) {
            StringBuffer pathSB = new StringBuffer(path);
            pathSB.append(Integer.toString(root.val));
            if (root.left == null && root.right == null) {  // 当前节点是叶子节点
                paths.add(pathSB.toString());  // 把路径加入到答案中
            } else {
                pathSB.append("->");  // 当前节点不是叶子节点，继续递归遍历
                constructPaths(root.left, pathSB.toString(), paths);
                constructPaths(root.right, pathSB.toString(), paths);
            }
        }
    }
}
```

- 时间复杂度：O(N^2)

- 空间复杂度：O(N^2)，最好情况下，当二叉树为平衡二叉树时，它的高度为 logN，此时空间复杂度为 O((logN)^2)

非递归

```java
    public List<String> binaryTreePaths(TreeNode root) {
        List<String> res = new ArrayList<>();
        if (root == null)
            return res;
        //栈中节点和路径都是成对出现的，路径表示的是从根节点到当前
        //节点的路径，如果到达根节点，说明找到了一条完整的路径
        Stack<Object> stack = new Stack<>();
        //当前节点和路径同时入栈
        stack.push(root);
        stack.push(root.val + "");
        while (!stack.isEmpty()) {
            //节点和路径同时出栈
            String path = (String) stack.pop();
            TreeNode node = (TreeNode) stack.pop();
            //如果是根节点，说明找到了一条完整路径，把它加入到集合中
            if (node.left == null && node.right == null) {
                res.add(path);
            }
            //右子节点不为空就把右子节点和路径压栈
            if (node.right != null) {
                stack.push(node.right);
                stack.push(path + "->" + node.right.val);
            }
            //左子节点不为空就把左子节点和路径压栈
            if (node.left != null) {
                stack.push(node.left);
                stack.push(path + "->" + node.left.val);
            }
        }
        return res;
    }
```



方法二：广度优先搜索

```java
class Solution {
    public List<String> binaryTreePaths(TreeNode root) {
        List<String> paths = new ArrayList<String>();
        if (root == null) {
            return paths;
        }
        Queue<TreeNode> nodeQueue = new LinkedList<TreeNode>();
        Queue<String> pathQueue = new LinkedList<String>();

        nodeQueue.offer(root);
        pathQueue.offer(Integer.toString(root.val));

        while (!nodeQueue.isEmpty()) {
            TreeNode node = nodeQueue.poll(); 
            String path = pathQueue.poll();

            if (node.left == null && node.right == null) {
                paths.add(path);
            } else {
                if (node.left != null) {
                    nodeQueue.offer(node.left);
                    pathQueue.offer(new StringBuffer(path).append("->").append(node.left.val).toString());
                }

                if (node.right != null) {
                    nodeQueue.offer(node.right);
                    pathQueue.offer(new StringBuffer(path).append("->").append(node.right.val).toString());
                }
            }
        }
        return paths;
    }
}
```

- 时间复杂度：O(N^2)

- 空间复杂度：O(N^2)

### [0147] 特殊数组的特征值：

给你一个非负整数数组 nums 。如果存在一个数 x ，使得 nums 中恰好有 x 个元素 大于或者等于 x ，那么就称 nums 是一个 特殊数组 ，而 x 是该数组的 特征值 。

注意： x 不必 是 nums 的中的元素。

如果数组 nums 是一个 特殊数组 ，请返回它的特征值 x 。否则，返回 -1 。可以证明的是，如果 nums 是特殊数组，那么其特征值 x 是 唯一的 。

 示例 1：

```
输入：nums = [3,5]
输出：2
解释：有 2 个元素（3 和 5）大于或等于 2 。
```

示例 2：

```
输入：nums = [0,0]
输出：-1
解释：没有满足题目要求的特殊数组，故而也不存在特征值 x 。
如果 x = 0，应该有 0 个元素 >= x，但实际有 2 个。
如果 x = 1，应该有 1 个元素 >= x，但实际有 0 个。
如果 x = 2，应该有 2 个元素 >= x，但实际有 0 个。
x 不能取更大的值，因为 nums 中只有两个元素。
```

示例 3：

```
输入：nums = [0,4,3,0,4]
输出：3
解释：有 3 个元素大于或等于 3 。
```

示例 4：

```
输入：nums = [3,6,7,7,0]
输出：-1
```


提示：

```
1 <= nums.length <= 100
0 <= nums[i] <= 1000
```

方法一：

```java
class Solution {
    public int specialArray(int[] nums) {
         if (nums == null || nums.length == 0) {
            return -1;
        }
        Arrays.sort(nums);
        int length = nums.length;
        //x最大只能是数组最大值或者数组元素个数
        int max = Math.max(nums[length - 1],length);
        //处理边界特殊情况
        if(nums[0] >= length) return length;
        for(int i = nums[0]; i <= max; ++i){
            if(((length - i -1)>=0 )&&((length - i)<length) &&(nums[length - i] >= i)&& nums[length - i -1]<i){
                return i;
            }
        }
        return -1;
    }
}

```

### [0148] 分式化简

 有一个同学在学习分式。他需要将一个连分数化成最简分数，你能帮助他吗？

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2019/09/09/fraction_example_1.jpg)

连分数是形如上图的分式。在本题中，所有系数都是大于等于0的整数。

 

输入的cont代表连分数的系数（cont[0]代表上图的a0，以此类推）。返回一个长度为2的数组[n, m]，使得连分数的值等于n / m，且n, m最大公约数为1。

 

示例 1：

```
输入：cont = [3, 2, 0, 2]
输出：[13, 4]
解释：原连分数等价于3 + (1 / (2 + (1 / (0 + 1 / 2))))。注意[26, 8], [-13, -4]都不是正确答案。
```


示例 2：

```
输入：cont = [0, 0, 3]
输出：[3, 1]
解释：如果答案是整数，令分母为1即可。
```


限制：

- cont[i] >= 0
- 1 <= cont的长度 <= 10
- cont最后一个元素不等于0
- 答案的n, m的取值都能被32位int整型存下（即不超过2 ^ 31 - 1）。

方法一：递归

```java
class Solution {
    public int[] recursive(int[] count, int index) {
        if (index == count.length - 1) {
            return new int[]{count[index], 1};
        }

        int[] nextRes = recursive(count, index+1);
        return new int[]{count[index] * nextRes[0] + nextRes[1], nextRes[0]};
    }

    public int[] fraction(int[] cont) {
        return recursive(cont, 0);
    }
}
```

方法二：

```java
class Solution {
    public int[] fraction(int[] cont) {
        int[] res = new int[2];
        res[0] = 1;
        res[1] = 0;
        for(int i = cont.length - 1; i >= 0; i--){
            int temp1 = res[1];
            res[1] = res[0];
            res[0] = cont[i] * res[1] + temp1;
        }
        return res;
    }
}
```

