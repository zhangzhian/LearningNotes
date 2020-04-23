# Leetcode数据结构与算法

### [0065]汉明距离

两个整数之间的汉明距离指的是这两个数字对应二进制位不同的位置的数目。

给出两个整数 x 和 y，计算它们之间的汉明距离。

注意：

```
0 ≤ x, y < 231.
```

示例:

```
输入: x = 1, y = 4

输出: 2

解释:
1   (0 0 0 1)
4   (0 1 0 0)
       ↑   ↑

上面的箭头指出了对应二进制位不同的位置。
```

方法一：异或

```java
class Solution {
    public int hammingDistance(int x, int y) {
        return countBitNumber(x^y);
    }
    //1.两个数字异或之后可以得到 不同二进制位的数字
    //2.计算该数字中1的个数，即是汉明距离
    //计算1的个数时，有几种方法，1：不断和左移的1 进行 与，判断该位是否为1.
    //                       2: n&(n-1)就是把n最右边的bit 1 位去掉，看能去掉几次，就有几个1位。(布赖恩·克尼根算法)
    int countBitNumber(int n)
    {
        int count = 0;
        while(n != 0)			
        {
            n = (n & n-1);
            ++count;
        }
				//while (n != 0) {
        //    if (n % 2 == 1)
        //       count += 1;
        //    n = n >> 1;
        //}
        return count;
    }
}
```

方法二：内置

```java
class Solution {
    public int hammingDistance(int x, int y) {
        return Integer.bitCount(x ^ y); 
    }
}
```

### [0066]将有序数组转换为二叉搜索树

将一个按照升序排列的有序数组，转换为一棵高度平衡二叉搜索树。

本题中，一个高度平衡二叉树是指一个二叉树每个节点 的左右两个子树的高度差的绝对值不超过 1。

示例:

给定有序数组: [-10,-3,0,5,9],

一个可能的答案是：[0,-3,9,-10,null,5]，它可以表示下面这个高度平衡二叉搜索树：

          0
         / \
       -3   9
       /   /
     -10  5
https://leetcode-cn.com/problems/convert-sorted-array-to-binary-search-tree/solution/jiang-you-xu-shu-zu-zhuan-huan-wei-er-cha-sou-s-15/

方法一：

```java
class Solution {
  int[] nums;

  public TreeNode helper(int left, int right) {
    if (left > right) return null;

    // always choose left middle node as a root
    int p = (left + right) / 2;

    // inorder traversal: left -> node -> right
    TreeNode root = new TreeNode(nums[p]);
    root.left = helper(left, p - 1);
    root.right = helper(p + 1, right);
    return root;
  }

  public TreeNode sortedArrayToBST(int[] nums) {
    this.nums = nums;
    return helper(0, nums.length - 1);
  }
}
```

### [0067]反转字符串中的单词 III

给定一个字符串，你需要反转字符串中每个单词的字符顺序，同时仍保留空格和单词的初始顺序。

示例 1:

```
输入: "Let's take LeetCode contest"
输出: "s'teL ekat edoCteeL tsetnoc" 
```


注意：在字符串中，每个单词由单个空格分隔，并且字符串中不会有任何额外的空格。

方法一：

```java
class Solution {
    public String reverseWords(String s) {
        String words[] = s.split(" ");
        StringBuilder res=new StringBuilder();
        for (String word: words)
            res.append(new StringBuffer(word).reverse().toString() + " ");
        return res.toString().trim();    
    }
}
```

方法二：

```java
public class Solution {
    public String reverseWords(String s) {
        String words[] = split(s);
        StringBuilder res=new StringBuilder();
        for (String word: words)
            res.append(reverse(word) + " ");
        return res.toString().trim();
    }
    public String[] split(String s) {
        ArrayList < String > words = new ArrayList < > ();
        StringBuilder word = new StringBuilder();
        for (int i = 0; i < s.length(); i++) {
            if (s.charAt(i) == ' ') {
                words.add(word.toString());
                word = new StringBuilder();
            } else
                word.append( s.charAt(i));
        }
        words.add(word.toString());
        return words.toArray(new String[words.size()]);
    }
    public String reverse(String s) {
      StringBuilder res=new StringBuilder();
        for (int i = 0; i < s.length(); i++)
            res.insert(0,s.charAt(i));
        return res.toString();
    }
}
```

方法三：

```java
public class Solution {
    public String reverseWords(String input) {
        final StringBuilder result = new StringBuilder();
        final StringBuilder word = new StringBuilder();
        for (int i = 0; i < input.length(); i++) {
            if (input.charAt(i) != ' ') {
                word.append(input.charAt(i));
            } else {
                result.append(word.reverse());
                result.append(" ");
                word.setLength(0);
            }
        }
        result.append(word.reverse());
        return result.toString();
    }
}
```

### [0068]非递增顺序的最小子序列

给你一个数组 nums，请你从中抽取一个子序列，满足该子序列的元素之和 严格 大于未包含在该子序列中的各元素之和。

如果存在多个解决方案，只需返回 长度最小 的子序列。如果仍然有多个解决方案，则返回 元素之和最大 的子序列。

与子数组不同的地方在于，「数组的子序列」不强调元素在原数组中的连续性，也就是说，它可以通过从数组中分离一些（也可能不分离）元素得到。

注意，题目数据保证满足所有约束条件的解决方案是 唯一 的。同时，返回的答案应当按 非递增顺序 排列。 

示例 1：

```
输入：nums = [4,3,10,9,8]
输出：[10,9] 
解释：子序列 [10,9] 和 [10,8] 是最小的、满足元素之和大于其他各元素之和的子序列。但是 [10,9] 的元素之和最大。 
```


示例 2：

```
输入：nums = [4,4,7,6,7]
输出：[7,7,6] 
解释：子序列 [7,7] 的和为 14 ，不严格大于剩下的其他元素之和（14 = 4 + 4 + 6）。因此，[7,6,7] 是满足题意的最小子序列。注意，元素按非递增顺序返回。  
```

示例 3：

```
输入：nums = [6]
输出：[6]
```

方法一：

先排序下，倒序遍历当前和是否大于剩余和，并去符合条件的元素

 ```java
class Solution {
        public List<Integer> minSubsequence(int[] nums) {
        List<Integer> res = new ArrayList<>();
        Arrays.sort(nums);
        int left = 0, sum = 0;
        for(int i : nums)
            left += i;
        for(int i = nums.length - 1; i >= 0; i--){
            sum += nums[i];
            left -= nums[i];
            res.add(nums[i]);
            if(sum > left)
                break;
        }
        return res;
    }
}
 ```



### [0069]山脉数组的峰顶索引

我们把符合下列属性的数组 A 称作山脉：

- A.length >= 3
- 存在 0 < i < A.length - 1 使得A[0] < A[1] < ... A[i-1] < A[i] > A[i+1] > ... > A[A.length - 1] 

给定一个确定为山脉的数组，返回任何满足 A[0] < A[1] < ... A[i-1] < A[i] > A[i+1] > ... > A[A.length - 1] 的 i 的值。

示例 1：

```
输入：[0,1,0]
输出：1
```


示例 2：

```
输入：[0,2,1,0]
输出：1
```

提示：

```
3 <= A.length <= 10000
0 <= A[i] <= 10^6
A 是如上定义的山脉
```

方法一：

```java
class Solution {
    public int peakIndexInMountainArray(int[] A) {
        int i = 0;
        while (A[i] < A[i+1]) i++;
        return i;
    }
}
```

方法二：二分查找

```java
class Solution {
    public int peakIndexInMountainArray(int[] A) {
        int lo = 0, hi = A.length - 1;
        while (lo < hi) {
            int mi = lo + (hi - lo) / 2;
            if (A[mi] < A[mi + 1])
                lo = mi + 1;
            else
                hi = mi;
        }
        return lo;
    }
}
```

### [0070]反转字符串

编写一个函数，其作用是将输入的字符串反转过来。输入字符串以字符数组 char[] 的形式给出。

不要给另外的数组分配额外的空间，你必须原地修改输入数组、使用 O(1) 的额外空间解决这一问题。

你可以假设数组中的所有字符都是 ASCII 码表中的可打印字符。

 示例 1：

```
输入：["h","e","l","l","o"]
输出：["o","l","l","e","h"]
```


示例 2：

```
输入：["H","a","n","n","a","h"]
输出：["h","a","n","n","a","H"]
```

方法一：双指针法

```java
class Solution {
    public void reverseString(char[] s) {
        int left = 0, right = s.length - 1;
        while (left < right) {
            char tmp = s[left];
            s[left++] = s[right];
            s[right--] = tmp;
        }
    }
}
```

方法二：

```java
class Solution {
    public void reverseString(char[] s) {
        for(int i = 0; i < s.length / 2; i++){
            char tmp = s[i];
            s[i] = s[s.length - 1 - i];
            s[s.length - 1 - i] = tmp;
        }
    }
}
```

### [0071]Nim 游戏

你和你的朋友，两个人一起玩 Nim 游戏：桌子上有一堆石头，每次你们轮流拿掉 1 - 3 块石头。 拿掉最后一块石头的人就是获胜者。你作为先手。

你们是聪明人，每一步都是最优解。 编写一个函数，来判断你是否可以在给定石头数量的情况下赢得游戏。

示例:

```
输入: 4
输出: false 
解释: 如果堆中有 4 块石头，那么你永远不会赢得比赛；
     因为无论你拿走 1 块、2 块 还是 3 块石头，最后一块石头总是会被你的朋友拿走。
```

方法：

```java
class Solution {
    public boolean canWinNim(int n) {
         return (n % 4 != 0);
    }
}
```



### [0072]可以被一步捕获的棋子数

在一个 8 x 8 的棋盘上，有一个白色的车（Rook），用字符 'R' 表示。棋盘上还可能存在空方块，白色的象（Bishop）以及黑色的卒（pawn），分别用字符 '.'，'B' 和 'p' 表示。不难看出，大写字符表示的是白棋，小写字符表示的是黑棋。

车按国际象棋中的规则移动。东，西，南，北四个基本方向任选其一，然后一直向选定的方向移动，直到满足下列四个条件之一：

棋手选择主动停下来。
棋子因到达棋盘的边缘而停下。
棋子移动到某一方格来捕获位于该方格上敌方（黑色）的卒，停在该方格内。
车不能进入/越过已经放有其他友方棋子（白色的象）的方格，停在友方棋子前。
你现在可以控制车移动一次，请你统计有多少敌方的卒处于你的捕获范围内（即，可以被一步捕获的棋子数）。

 示例 1：

![](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2019/02/23/1253_example_1_improved.PNG)

```
输入：[[".",".",".",".",".",".",".","."],
[".",".",".","p",".",".",".","."],
[".",".",".","R",".",".",".","p"],
[".",".",".",".",".",".",".","."],
[".",".",".",".",".",".",".","."],
[".",".",".","p",".",".",".","."],
[".",".",".",".",".",".",".","."],
[".",".",".",".",".",".",".","."]]
输出：3
解释：
在本例中，车能够捕获所有的卒。
```

示例 2：

![](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2019/02/23/1253_example_2_improved.PNG)

```
输入：[[".",".",".",".",".",".",".","."],
[".","p","p","p","p","p",".","."],
[".","p","p","B","p","p",".","."],
[".","p","B","R","B","p",".","."],
[".","p","p","B","p","p",".","."],
[".","p","p","p","p","p",".","."],
[".",".",".",".",".",".",".","."],
[".",".",".",".",".",".",".","."]]
输出：0
解释：
象阻止了车捕获任何卒。
```

示例 3：

![](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2019/02/23/1253_example_3_improved.PNG)

```
输入：[[".",".",".",".",".",".",".","."],
[".",".",".","p",".",".",".","."],
[".",".",".","p",".",".",".","."],
["p","p",".","R",".","p","B","."],
[".",".",".",".",".",".",".","."],
[".",".",".","B",".",".",".","."],
[".",".",".","p",".",".",".","."],
[".",".",".",".",".",".",".","."]]
输出：3
解释： 
车可以捕获位置 b5，d6 和 f5 的卒。
```


提示：

board.length == board[i].length == 8
board[i][j] 可以是 'R'，'.'，'B' 或 'p'
只有一个格子上存在 board[i][j] == 'R'

方法一：

https://leetcode-cn.com/problems/available-captures-for-rook/solution/mo-ni-ti-an-zhao-ti-mu-yi-si-shi-xian-ji-ke-java-b/

```java
class Solution {
    public int numRookCaptures(char[][] board) {
        // 因为题目已经明确给出 board.length == board[i].length == 8，所以不做输入检查
        // 定义方向数组，可以认为是四个方向向量，在棋盘问题上是常见的做法
        int[][] directions = {{-1, 0}, {1, 0}, {0, 1}, {0, -1}};

        for (int i = 0; i < 8; i++) {
            for (int j = 0; j < 8; j++) {              
                if (board[i][j] == 'R') {
                    int res = 0;
                    for (int[] direction : directions) {
                        if (burnout(board, i, j, direction)) {
                            res++;
                        }
                    }
                    return res;
                }
            }
        }
        // 代码不会走到这里，返回 0 或者抛出异常均可
        return 0;
    }

    /**
     * burnout 横冲直撞的意思（来自欧路词典）
     *
     * @param board     输入棋盘
     * @param x         当前白象位置的横坐标
     * @param y         当前白象位置的纵坐标
     * @param direction 方向向量
     * @return 消灭一个 p，就返回 true
     */
    private boolean burnout(char[][] board, int x, int y, int[] direction) {
        int i = x;
        int j = y;
        while (inArea(i, j)) {
            // 是友军，路被堵死，直接返回
            if (board[i][j] == 'B') {
                break;
            }

            // 是敌军，拿下一血（不知道一血这个词是不是这么用的）
            if (board[i][j] == 'p') {
                return true;
            }

            i += direction[0];
            j += direction[1];
        }
        return false;
    }

    /**
     * @param i 当前位置横坐标
     * @param j 当前位置纵坐标
     * @return 是否在棋盘有效范围内
     */
    private boolean inArea(int i, int j) {
        return i >= 0 && i < 8 && j >= 0 && j < 8;
    }
}
```

### [0073]从上到下打印二叉树 II

从上到下按层打印二叉树，同一层的节点按从左到右的顺序打印，每一层打印到一行。

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
  [9,20],
  [15,7]
]
```


提示：

节点总数 <= 1000

方法一：队列

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
            List<Integer> tmp = new ArrayList<>();
            for(int i = queue.size(); i > 0; i--) {
                TreeNode node = queue.poll();
                tmp.add(node.val);
                if(node.left != null) queue.add(node.left);
                if(node.right != null) queue.add(node.right);
            }
            res.add(tmp);
        }
        return res;     
    }
}
```

方法二：递归

```java
class Solution {
    List<List<Integer>> res = new ArrayList();
    public List<List<Integer>> levelOrder(TreeNode root){
        recur(root,0);
        return res;
    }
    public void recur(TreeNode root,int k){
        if(root != null){
            if(res.size() <= k)
                res.add(new ArrayList());
            res.get(k).add(root.val);
            recur(root.left,k+1);
            recur(root.right,k+1);
        }
    }
}
```

### [0074]逐步求和得到正数的最小值

给你一个整数数组 nums 。你可以选定任意的 正数 startValue 作为初始值。

你需要从左到右遍历 nums 数组，并将 startValue 依次累加上 nums 数组中的值。

请你在确保累加和始终大于等于 1 的前提下，选出一个最小的 正数 作为 startValue 。

 示例 1：

```
输入：nums = [-3,2,-3,4,2]
输出：5
解释：如果你选择 startValue = 4，在第三次累加时，和小于 1 。
                累加求和
                startValue = 4 | startValue = 5 | nums
                  (4 -3 ) = 1  | (5 -3 ) = 2    |  -3
                  (1 +2 ) = 3  | (2 +2 ) = 4    |   2
                  (3 -3 ) = 0  | (4 -3 ) = 1    |  -3
                  (0 +4 ) = 4  | (1 +4 ) = 5    |   4
                  (4 +2 ) = 6  | (5 +2 ) = 7    |   2
```


示例 2：

```
输入：nums = [1,2]
输出：1
解释：最小的 startValue 需要是正数。
```


示例 3：

```
输入：nums = [1,-2,-3]
输出：5
```


提示：

1 <= nums.length <= 100
-100 <= nums[i] <= 100

方法一：

找一个开始值，然后以这个值开始加数组中的每一个数，中间每一步都要大于 1。

加每一个数就是前缀和。

如果要每一步都大于 1 ，只需要找到前缀和中最小的值。

```java
class Solution {
    public int minStartValue(int[] nums) {
        int sum = 0;
        int min = Integer.MAX_VALUE;
        for (int i = 0; i < nums.length; i++) {
            sum += nums[i];
            if (sum < min) {
                min = sum;
            }
        }
        return Math.max(1, 1 - min);
    }
}
```

### [0075]拼写单词

给你一份『词汇表』（字符串数组） words 和一张『字母表』（字符串） chars。

假如你可以用 chars 中的『字母』（字符）拼写出 words 中的某个『单词』（字符串），那么我们就认为你掌握了这个单词。

注意：每次拼写（指拼写词汇表中的一个单词）时，chars 中的每个字母都只能用一次。

返回词汇表 words 中你掌握的所有单词的 长度之和。

示例 1：

```
输入：words = ["cat","bt","hat","tree"], chars = "atach"
输出：6
解释： 
可以形成字符串 "cat" 和 "hat"，所以答案是 3 + 3 = 6。
```


示例 2：

```
输入：words = ["hello","world","leetcode"], chars = "welldonehoneyr"
输出：10
解释：
可以形成字符串 "hello" 和 "world"，所以答案是 5 + 5 = 10。
```

提示：

```
1 <= words.length <= 1000
1 <= words[i].length, chars.length <= 100
所有字符串中都仅包含小写英文字母
```

方法一：

```java
class Solution {
    public int countCharacters(String[] words, String chars) {
        int[] chars_count = count(chars); // 统计字母表的字母出现次数
        int res = 0;
        for (String word : words) {
            int[] word_count = count(word); // 统计单词的字母出现次数
            if (contains(chars_count, word_count)) {
                res += word.length();
            }
        }
        return res;
    }

    // 检查字母表的字母出现次数是否覆盖单词的字母出现次数
    boolean contains(int[] chars_count, int[] word_count) {
        for (int i = 0; i < 26; i++) {
            if (chars_count[i] < word_count[i]) {
                return false;
            }
        }
        return true;
    }

    // 统计 26 个字母出现的次数
    int[] count(String word) {
        int[] counter = new int[26];
        for (int i = 0; i < word.length(); i++) {
            char c = word.charAt(i);
            counter[c-'a']++;
        }
        return counter;
    }
}
```

### [0076]N叉树的最大深度

给定一个 N 叉树，找到其最大深度。

最大深度是指从根节点到最远叶子节点的最长路径上的节点总数。

例如，给定一个 3叉树 :

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/12/narytreeexample.png)  

我们应返回其最大深度，3。

说明:

树的深度不会超过 1000。
树的节点总不会超过 5000。

方法一：递归

```java
/*
// Definition for a Node.
class Node {
    public int val;
    public List<Node> children;

    public Node() {}

    public Node(int _val) {
        val = _val;
    }

    public Node(int _val, List<Node> _children) {
        val = _val;
        children = _children;
    }
};
*/
class Solution {
  public int maxDepth(Node root) {
    if (root == null) {
      return 0;
    } else if (root.children.isEmpty()) {
      return 1;  
    } else {
      List<Integer> heights = new LinkedList<>();
      for (Node item : root.children) {
        heights.add(maxDepth(item)); 
      }
      return Collections.max(heights) + 1;
    }
  }
}
```

方法二: 迭代

```java
class Solution {
  public int maxDepth(Node root) {
    Queue<Pair<Node, Integer>> stack = new LinkedList<>();
    if (root != null) {
      stack.add(new Pair(root, 1));
    }

    int depth = 0;
    while (!stack.isEmpty()) {
      Pair<Node, Integer> current = stack.poll();
      root = current.getKey();
      int current_depth = current.getValue();
      if (root != null) {
        depth = Math.max(depth, current_depth);
        for (Node c : root.children) {
          stack.add(new Pair(c, current_depth + 1));    
        }
      }
    }
    return depth;
  }
}
```

### [0077]删列造序

给定由 N 个小写字母字符串组成的数组 A，其中每个字符串长度相等。

你需要选出一组要删掉的列 D，对 A 执行删除操作，使 A 中剩余的每一列都是 非降序 排列的，然后请你返回 D.length 的最小可能值。

删除 操作的定义是：选出一组要删掉的列，删去 A 中对应列中的所有字符，形式上，第 n 列为` [A[0][n], A[1][n], ..., A[A.length-1][n]]`。

比如，有` A = ["abcdef", "uvwxyz"]`，

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2019/07/06/944_1.png)

要删掉的列为 {0, 2, 3}，删除后 A 为["bef", "vyz"]， A 的列分别为["b","v"], ["e","y"], ["f","z"]。

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2019/07/06/944_2.png)

示例 1：

```
输入：["cba", "daf", "ghi"]
输出：1
解释：
当选择 D = {1}，删除后 A 的列为：["c","d","g"] 和 ["a","f","i"]，均为非降序排列。
若选择 D = {}，那么 A 的列 ["b","a","h"] 就不是非降序排列了。
```


示例 2：

```
输入：["a", "b"]
输出：0
解释：D = {}
```


示例 3：

```
输入：["zyx", "wvu", "tsr"]
输出：3
解释：D = {0, 1, 2}
```

提示：

```
1 <= A.length <= 100
1 <= A[i].length <= 1000
```

方法一：

```java
class Solution {
    public int minDeletionSize(String[] A) {
        int ans = 0;
        for (int c = 0; c < A[0].length(); ++c)
            for (int r = 0; r < A.length - 1; ++r)
                if (A[r].charAt(c) > A[r+1].charAt(c)) {
                    ans++;
                    break;
                }

        return ans;
    }
}
```



