# Leetcode数据结构与算法

### [0129] 二叉树的层次遍历 II

给定一个二叉树，返回其节点值自底向上的层次遍历。 （即按从叶子节点所在层到根节点所在的层，逐层从左向右遍历）

例如：
给定二叉树 [3,9,20,null,null,15,7],

    	3
       / \
      9  20
        /  \
       15   7

返回其自底向上的层次遍历为：

```
[
  [15,7],
  [9,20],
  [3]
]
```

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
    public List<List<Integer>> levelOrderBottom(TreeNode root) {
       if(root==null) {
            return new ArrayList<List<Integer>>();
        }
        //用来存放最终结果
        ArrayList<List<Integer>> res = new ArrayList<List<Integer>>();
        //创建一个队列，将根节点放入其中
        Queue<TreeNode> queue = new LinkedList<TreeNode>();
        queue.offer(root);
        while(!queue.isEmpty()) {
            //每次遍历的数量为队列的长度
            int size = queue.size();
            ArrayList<Integer> tmp = new ArrayList<Integer>();
            //将这一层的元素全部取出，放入到临时数组中，如果节点的左右孩子不为空，继续放入队列
            for(int i=0;i<size;++i) {
                TreeNode node = queue.poll();
                tmp.add(node.val);
                if(node.left!=null) {
                    queue.offer(node.left);
                }
                if(node.right!=null) {
                    queue.offer(node.right);
                }
            }
            //直接添加到头部
            res.add(0,tmp);
        }
        return res;
    }     
}
```

方法二：DFS

```java
class Solution {
    public List<List<Integer>> levelOrderBottom(TreeNode root) {
        if(root==null) {
            return new ArrayList<List<Integer>>();
        }
        //用来存放最终结果
        ArrayList<List<Integer>> res = new ArrayList<List<Integer>>();
        dfs(root,res,1);
        Collections.reverse(res);
        return res;
    }

    private void dfs(TreeNode root,ArrayList<List<Integer>> res,int index) {
        if(root==null) {
            return;
        }
        //如果index大于res大小，说明这一层没有对应的集合，需要新创建
        if(index>res.size()) {
            res.add(new ArrayList<Integer>());
        }
        //将当前层的元素直接放到对应层的末尾即可
        res.get(index-1).add(root.val);
        //继续遍历左右节点，同时将层高+1
        dfs(root.left,res,index+1);
        dfs(root.right,res,index+1);
    }
}
```

### [0130] 杨辉三角

给定一个非负整数 numRows，生成杨辉三角的前 numRows 行。

在杨辉三角中，每个数是它左上方和右上方的数的和。

示例:

![img](https://upload.wikimedia.org/wikipedia/commons/0/0d/PascalTriangleAnimated2.gif)

输入: 5
输出:
[
     [1],
    [1,1],
   [1,2,1],
  [1,3,3,1],
 [1,4,6,4,1]
]

方法一：动态规划

```java
class Solution {
    public List<List<Integer>> generate(int numRows) {
        List<List<Integer>> triangle = new ArrayList<List<Integer>>();

        // First base case; if user requests zero rows, they get zero rows.
        if (numRows == 0) {
            return triangle;
        }

        // Second base case; first row is always [1].
        triangle.add(new ArrayList<>());
        triangle.get(0).add(1);

        for (int rowNum = 1; rowNum < numRows; rowNum++) {
            List<Integer> row = new ArrayList<>();
            List<Integer> prevRow = triangle.get(rowNum-1);

            // The first row element is always 1.
            row.add(1);

            for (int j = 1; j < rowNum; j++) {
                row.add(prevRow.get(j-1) + prevRow.get(j));
            }

            // The last row element is always 1.
            row.add(1);

            triangle.add(row);
        }

        return triangle;
    }
}
```

方法二：递归

```java
class Solution {
    public List<List<Integer>> generate(int numRows) {
    	//存储要返回的杨辉三角
        List<List<Integer>> triangle = new ArrayList<>();
        //若0行，则返回空
        if(numRows == 0){
            return triangle;
        }
        //递归出口，第一步找到出口
        if(numRows == 1){
            triangle.add(new ArrayList<>());
            triangle.get(0).add(1);
            return triangle;
        }
        //递归，注意返回值这是第二步
        triangle = generate(numRows-1);
        //要做的第三步
        List<Integer> row = new ArrayList<>();
        List<Integer> prevRow = triangle.get(numRows-2);
        row.add(1);
        for(int j = 1;j < numRows-1;j++){
            row.add(prevRow.get(j-1) + prevRow.get(j));
        }
        row.add(1);
        triangle.add(row);
        return triangle;
    }
}
```

### [0131] 文件夹操作日志搜集器

每当用户执行变更文件夹操作时，LeetCode 文件系统都会保存一条日志记录。

下面给出对变更操作的说明：

"../" ：移动到当前文件夹的父文件夹。如果已经在主文件夹下，则 继续停留在当前文件夹 。
"./" ：继续停留在当前文件夹。
"x/" ：移动到名为 x 的子文件夹中。题目数据 保证总是存在文件夹 x 。
给你一个字符串列表 logs ，其中 logs[i] 是用户在 ith 步执行的操作。

文件系统启动时位于主文件夹，然后执行 logs 中的操作。

执行完所有变更文件夹操作后，请你找出 返回主文件夹所需的最小步数 。

 ![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/09/26/sample_11_1957.png)

```
输入：logs = ["d1/","d2/","../","d21/","./"]
输出：2
解释：执行 "../" 操作变更文件夹 2 次，即可回到主文件夹
```

**示例 2：**

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/09/26/sample_22_1957.png)

```
输入：logs = ["d1/","d2/","./","d3/","../","d31/"]
输出：3
```

示例 3：

```
输入：logs = ["d1/","../","../","../"]
输出：0
```

提示：

- 1 <= logs.length <= 103
- 2 <= logs[i].length <= 10
- logs[i] 包含小写英文字母，数字，'.' 和 '/'
- logs[i] 符合语句中描述的格式
- 文件夹名称由小写英文字母和数字组成

方法一：直接遍历

```java
class Solution {
    public int minOperations(String[] logs) {
        int len = logs.length,res = 0;
        for(int i=0;i<len;i++){
            if(logs[i].equals("../")){
                if(res!=0)
                    res--;
            }else if(logs[i].equals("./")){
                continue;
            }else {
                res++;
            }
        }
        return res;
    }
}
```

方法二：栈

```java
class Solution {
    public int minOperations(String[] logs) {
        Stack<String> stack = new Stack<>();
        for (String log : logs) {
            if (log.equals("../")) {
                if (!stack.isEmpty()) {
                    stack.pop();
                }
            } else if ( log.equals("./")){

            } else {
                stack.push(log);
            }
        }
        return stack.size();
    }
}
```

### [0132] 根据数字二进制下 1 的数目排序

给你一个整数数组 arr 。请你将数组中的元素按照其二进制表示中数字 1 的数目升序排序。

如果存在多个数字二进制中 1 的数目相同，则必须将它们按照数值大小升序排列。

请你返回排序后的数组。

示例 1：

```
输入：arr = [0,1,2,3,4,5,6,7,8]
输出：[0,1,2,4,8,3,5,6,7]
解释：[0] 是唯一一个有 0 个 1 的数。
[1,2,4,8] 都有 1 个 1 。
[3,5,6] 有 2 个 1 。
[7] 有 3 个 1 。
按照 1 的个数排序得到的结果数组为 [0,1,2,4,8,3,5,6,7]
```

示例 2：

```
输入：arr = [1024,512,256,128,64,32,16,8,4,2,1]
输出：[1,2,4,8,16,32,64,128,256,512,1024]
解释：数组中所有整数二进制下都只有 1 个 1 ，所以你需要按照数值大小将它们排序。
```

示例 3：

```
输入：arr = [10000,10000]
输出：[10000,10000]
```

示例 4：

```
输入：arr = [2,3,5,7,11,13,17,19]
输出：[2,3,5,17,7,11,13,19]
```

示例 5：

```
输入：arr = [10,100,1000,10000]
输出：[10,100,10000,1000]
```


提示：

- 1 <= arr.length <= 500
- 0 <= arr[i] <= 10^4

方法一：

循环并使用Integer.bitCount计算数字中1的个数，乘以100000（题目中不会大于10^4）然后加上原数字，放入数组map中，并对map进行排序，最后% 100000获取原来的数组，填充到原数组返回即可

```java
class Solution {
    public int[] sortByBits(int[] arr) {
        int[] map = new int[arr.length];
        for (int i = 0; i < arr.length; i++) {
            map[i] = Integer.bitCount(arr[i]) * 100000 + arr[i];
        }
        Arrays.sort(map);
        for (int i = 0; i < map.length; i++) {
            map[i] = map[i] % 100000;
        }
        return map;
    }
}
```

方法二：

```java
class Solution {
    public int[] sortByBits(int[] arr) {
        quickSort(arr, 0, arr.length - 1);
        return arr;
    }
    private void quickSort(int[] arr, int lo, int hi) {
        if (lo >= hi) {
            return;
        }
        // 每次切分出来一个排好序的元素的索引 cut
        // arr[cut] 左边的元素都不大于他， 右边的元素都不小于他
        int cut = partition(arr, lo, hi);
        quickSort(arr, lo, cut - 1);
        quickSort(arr, cut + 1, hi);
    }

    private int partition(int[] arr, int lo, int hi) {
        // 选择一个标记元素，一般选最左边或者最右边
        int temp = arr[hi];
        // 两个索引， cut 和 i
        int cut = lo;
        // 如果 arr[i] 比 temp 小的话，交换 arr[i] 和 arr[cut], 然后 cut++
        // 切分的实质就是把 temp 放到数组中合适的位置，该位置的索引就是 cut
        for (int i = lo; i < hi; i++) {
            // 如果二进制 1 的个数相等，则比较两个数的实际大小
            if (countBits(arr[i]) == countBits(temp)) {
              if (arr[i]  < temp) {
                  swap(arr, i, cut);
                  cut++;
              }
            } else if (countBits(arr[i]) < countBits(temp)) {
                swap(arr, i, cut);
                cut++;
            }
        }
        swap(arr, cut, hi);
        return cut;
    }
    public void swap(int[] arr, int i, int j) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
    
    //对于一个二进制数 n
    //n & (n - 1) 可以把 n 的最后一位 1 变成 0
    //因为 n 和 n - 1 最后一位肯定不同。
    public int countBits(int num) {
        int count = 0;
        while (num != 0) {
            count++;
            num = num & (num - 1);
        }
        return count;
    }
}
```

### [0133] 数组中重复的数字

找出数组中重复的数字。


在一个长度为 n 的数组 nums 里的所有数字都在 0～n-1 的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。

示例 1：

```
输入：
[2, 3, 1, 0, 2, 5, 3]
输出：2 或 3 
```


限制：

2 <= n <= 100000

方法一：遍历数组

```java
class Solution {
    public int findRepeatNumber(int[] nums) {
        Set<Integer> set = new HashSet<Integer>();
        int repeat = -1;
        for (int num : nums) {
            if (!set.add(num)) {
                repeat = num;
                break;
            }
        }
        return repeat;
    }
}
```

时间复杂度：O(n)

空间复杂度：O(n)

方法二：

```java
class Solution {
    public int findRepeatNumber(int[] nums) {
        int temp;
        for(int i=0;i<nums.length;i++){
            while (nums[i]!=i){
                if(nums[i]==nums[nums[i]]){
                    return nums[i];
                }
                temp=nums[i];
                nums[i]=nums[temp];
                nums[temp]=temp;
            }
        }
        return -1;
    }
}
```

时间复杂度：O(n)

空间复杂度：O(1)

### [0134] 最小绝对差

给你个整数数组 arr，其中每个元素都 不相同。

请你找到所有具有最小绝对差的元素对，并且按升序的顺序返回。

 示例 1：

```
输入：arr = [4,2,1,3]
输出：[[1,2],[2,3],[3,4]]
```

示例 2：

```
输入：arr = [1,3,6,10,15]
输出：[[1,3]]
```

示例 3：

```
输入：arr = [3,8,-10,23,19,-4,-14,27]
输出：[[-14,-10],[19,23],[23,27]]
```


提示：

```
2 <= arr.length <= 10^5
-10^6 <= arr[i] <= 10^6
```

方法一：

```java
class Solution {
    public List<List<Integer>> minimumAbsDifference(int[] arr) {
        Arrays.sort(arr);
        List<List<Integer>> res = new LinkedList<>();
        int min = arr[1] - arr[0];
        for (int i = 1; i < arr.length; i++) {
            int tmp = arr[i] - arr[i - 1];
            // 找到新的最小差，清空原有结果
            if (tmp < min) {
                min = tmp;
                res.clear();
            }
            // 如果是最小差，记录
            if (tmp == min) {
                List<Integer> item = new ArrayList<>(2);
                item.add(arr[i - 1]);
                item.add(arr[i]);
                res.add(item);
            }
        }
        return res;
    }
}
```

### [0135] 二进制矩阵中的特殊位置

给你一个大小为 rows x cols 的矩阵 mat，其中 mat[i][j] 是 0 或 1，请返回 矩阵 mat 中特殊位置的数目 。

特殊位置 定义：如果 mat[i][j] == 1 并且第 i 行和第 j 列中的所有其他元素均为 0（行和列的下标均 从 0 开始 ），则位置 (i, j) 被称为特殊位置。

示例 1：

```
输入：mat = [[1,0,0],
            [0,0,1],
            [1,0,0]]
输出：1
解释：(1,2) 是一个特殊位置，因为 mat[1][2] == 1 且所处的行和列上所有其他元素都是 0
```

示例 2：

```
输入：mat = [[1,0,0],
            [0,1,0],
            [0,0,1]]
输出：3
解释：(0,0), (1,1) 和 (2,2) 都是特殊位置
```

示例 3：

```
输入：mat = [[0,0,0,1],
            [1,0,0,0],
            [0,1,1,0],
            [0,0,0,0]]
输出：2
```

示例 4：

```
输入：mat = [[0,0,0,0,0],
            [1,0,0,0,0],
            [0,1,0,0,0],
            [0,0,1,0,0],
            [0,0,0,1,1]]
输出：3
```


提示：

- rows == mat.length
- cols == mat[i].length
- 1 <= rows, cols <= 100
- `mat[i][j]` 是 0 或 1

方法一：暴力

```java
class Solution {
    public int numSpecial(int[][] mat) {
        //存储行和列的和
        int[] rows=new int[mat.length];
        int[] cols=new int[mat[0].length];
        int cut=0;
        for(int i=0;i<mat.length;i++){
            for(int j=0;j<mat[0].length;j++){
                rows[i]+=mat[i][j];
                cols[j]+=mat[i][j];
            }
        }
        for(int i=0;i<mat.length;i++){
            for(int j=0;j<mat[0].length;j++){
                //当前为1并且行列和均为1，即可确定是特殊位置
                if(mat[i][j]==1 && rows[i]==1 && cols[j]==1 ) {
                    cut++;
                    //一行不存在两个特殊位置
                    break;
                }
            }
        }
        return cut;
    }
}
```

### [0136] 修剪二叉搜索树

给定一个二叉搜索树，同时给定最小边界L 和最大边界 R。通过修剪二叉搜索树，使得所有节点的值在[L, R]中 (R>=L) 。你可能需要改变树的根节点，所以结果应当返回修剪好的二叉搜索树的新的根节点。

示例 1:

```
输入: 
    1
   / \
  0   2

  L = 1
  R = 2

输出: 
    1
      \
       2
```

示例 2:

```
输入: 
    3
   / \
  0   4
   \
    2
   /
  1

  L = 1
  R = 3

输出: 
      3
     / 
   2   
  /
 1
```

方法一：递归

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
 class Solution {
    public TreeNode trimBST(TreeNode root, int L, int R) {
        if (root == null) return root;
        //因为是二叉搜索树,节点.left < 节点 < 节点.right
        //如果数字比right大,就把右节点全部裁掉.
        //裁掉之后,继续看左节点的剪裁情况
        if (root.val > R) return trimBST(root.left, L, R);
        //节点数字比left小,就把左节点全部裁掉.
        //裁掉之后,继续看右节点的剪裁情况.剪裁后重新赋值给root.
        if (root.val < L) return trimBST(root.right, L, R);

        root.left = trimBST(root.left, L, R);
        root.right = trimBST(root.right, L, R);
        return root;
    }
}

```

### [0137] 数组的相对排序

给你两个数组，arr1 和 arr2，

- arr2 中的元素各不相同
- arr2 中的每个元素都出现在 arr1 中

对 arr1 中的元素进行排序，使 arr1 中项的相对顺序和 arr2 中的相对顺序相同。未在 arr2 中出现过的元素需要按照升序放在 arr1 的末尾。

示例：

```
输入：arr1 = [2,3,1,3,2,4,6,7,9,2,19], arr2 = [2,1,4,3,9,6]
输出：[2,2,2,1,4,3,3,9,6,7,19]
```


提示：

- arr1.length, arr2.length <= 1000
- 0 <= arr1[i], arr2[i] <= 1000
- arr2 中的元素 arr2[i] 各不相同
- arr2 中的每个元素 arr2[i] 都出现在 arr1 中

方法一：计数排序

```java
class Solution {
    public int[] relativeSortArray(int[] arr1, int[] arr2) {
        int[] count = new int[1001];
        // 将 arr1 中的数记录下来
        for (int num1 : arr1) {
            count[num1]++;
        }
        // 先安排 arr2 中的数
        int i = 0;
        for (int num2 : arr2) {
            while (count[num2] > 0) {
                arr1[i++] = num2;
                count[num2]--;
            }
        }
        // 再排剩下的数
        for (int num1 = 0; num1 < count.length; num1++) {
            while (count[num1] > 0) {
                arr1[i++] = num1;
                count[num1]--;
            }
        }
        return arr1;
    }
}
```

时间复杂度：O(n+m)

空间复杂度：O(n)

方法二：

自定义比较函数，基于比较函数进行排序

```java
Map<Integer, Integer> record = new HashMap<>(arr2.length);
for (int i = 0; i < arr2.length; i++) {
    record.put(arr2[i], i);
}
public boolean less(int num1, int num2) {
    if (record.containsKey(num1) && record.containsKey(num2)) {
        return record.get(num1) < record.get(num2);
    } else if (record.containsKey(num1)) {
        return true;
    } else if (record.containsKey(num2)) {
        return false;
    } else {
        return num1 < num2;
    }
}
```

### [0138] 从根到叶的二进制数之和

给出一棵二叉树，其上每个结点的值都是 0 或 1 。每一条从根到叶的路径都代表一个从最高有效位开始的二进制数。例如，如果路径为 0 -> 1 -> 1 -> 0 -> 1，那么它表示二进制数 01101，也就是 13 。

对树上的每一片叶子，我们都要找出从根到该叶子的路径所表示的数字。

以 10^9 + 7 为模，返回这些数字之和。

示例：

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2019/04/05/sum-of-root-to-leaf-binary-numbers.png)

输入：[1,0,1,0,1,0,1]
输出：22
解释：(100) + (101) + (110) + (111) = 4 + 5 + 6 + 7 = 22


提示：

- 树中的结点数介于 1 和 1000 之间。
- node.val 为 0 或 1 。

方法一：BFS写法

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public int sumRootToLeaf(TreeNode root) {
        if(root == null){
            return 0;
        }
        int res = 0;
        Queue<TreeNode> nodeQueue = new LinkedList<>();
        Queue<Integer> queue = new LinkedList<>();
        nodeQueue.add(root);
        queue.add(root.val);
        while(!nodeQueue.isEmpty()){
            // 同时维护两个队列
            TreeNode node = nodeQueue.poll();
            int tmp = queue.poll();

            // 如果该节点是叶子节点，加到res中
            if(node.left==null && node.right==null){
                res += tmp;
            } else {
                // 左节点不为空时，左节点进入队列，左节点对应的值是当前节点tmp<<1+node.left.val
                if(node.left != null){
                    nodeQueue.add(node.left);
                    queue.add((tmp<<1) + node.left.val);
                }
                if(node.right != null){
                    nodeQueue.add(node.right);
                    queue.add((tmp<<1) + node.right.val);
                }
            }
        }
        return res;
    } 
}
```

方法二：DFS写法

```java
class Solution {
    int res = 0;

    public int sumRootToLeaf(TreeNode root){
        preOrder(root,0);
        return res;
    }
    public void preOrder(TreeNode root, int val){
        if(root != null){
            // 值先移位，后相加
            int tmp = (val<<1) + root.val;
                
            // 当前节点是叶子节点
            if(root.left == null && root.right == null){
                res += tmp;
            } else {
                // 当前节点的左子节点不为空，继续递归，val的值是父节点的值，也就是tmp
                if(root.left != null){
                    preOrder(root.left, tmp);
                }
                if(root.right != null){
                    preOrder(root.right, tmp);
                }
            }
        }
    }
}
```

### [0139] 重复 N 次的元素

在大小为 2N 的数组 A 中有 N+1 个不同的元素，其中有一个元素重复了 N 次。

返回重复了 N 次的那个元素。

示例 1：

```
输入：[1,2,3,3]
输出：3
```

示例 2：

```
输入：[2,1,2,5,3,2]
输出：2
```


示例 3：

```
输入：[5,1,5,2,5,3,5,4]
输出：5
```


提示：

- 4 <= A.length <= 10000
- 0 <= A[i] < 10000
- A.length 为偶数

方法一：计数

```java
class Solution {
    public int repeatedNTimes(int[] A) {
        Map<Integer, Integer> count = new HashMap();
        for (int x: A) {
            count.put(x, count.getOrDefault(x, 0) + 1);
        }

        for (int k: count.keySet())
            if (count.get(k) > 1)
                return k;

        throw null;
    }
}
```

- 时间复杂度：O(N)
- 空间复杂度：O(N)

方法二：比较

```java
class Solution {
    public int repeatedNTimes(int[] A) {
        for (int k = 1; k <= 3; ++k)
            for (int i = 0; i < A.length - k; ++i)
                if (A[i] == A[i+k])
                    return A[i];

        throw null;
    }
}
```

- 时间复杂度：O(N)
- 空间复杂度：O(1)

### [0140] 找出数组中的幸运数

在整数数组中，如果一个整数的出现频次和它的数值大小相等，我们就称这个整数为「幸运数」。

给你一个整数数组 arr，请你从中找出并返回一个幸运数。

如果数组中存在多个幸运数，只需返回 最大 的那个。
如果数组中不含幸运数，则返回 -1 。


示例 1：

```
输入：arr = [2,2,3,4]
输出：2
解释：数组中唯一的幸运数是 2 ，因为数值 2 的出现频次也是 2 。
```

示例 2：

```
输入：arr = [1,2,2,3,3,3]
输出：3
解释：1、2 以及 3 都是幸运数，只需要返回其中最大的 3 。
```

示例 3：

```
输入：arr = [2,2,2,3,3]
输出：-1
解释：数组中不存在幸运数。
```

示例 4：

```
输入：arr = [5]
输出：-1
```

示例 5：

```
输入：arr = [7,7,7,7,7,7,7]
输出：7
```


提示：

1 <= arr.length <= 500
1 <= arr[i] <= 500

方法一：

```java
class Solution {
    public int findLucky(int[] arr) {
        int n = arr.length;
        int[] map = new int[501];
        for(int num : arr) map[num]++;
        for(int i=500; i>0; i--) 
            if(i==map[i])
                return i;

        return -1;
    }
}
```

### [0141] 托普利茨矩阵

如果矩阵上每一条由左上到右下的对角线上的元素都相同，那么这个矩阵是 托普利茨矩阵 。

给定一个 M x N 的矩阵，当且仅当它是托普利茨矩阵时返回 True。

示例 1:

```
输入: 
matrix = [
  [1,2,3,4],
  [5,1,2,3],
  [9,5,1,2]
]
输出: True
解释:
在上述矩阵中, 其对角线为:
"[9]", "[5, 5]", "[1, 1, 1]", "[2, 2, 2]", "[3, 3]", "[4]"。
各条对角线上的所有元素均相同, 因此答案是True。
```

示例 2:

```
输入:
matrix = [
  [1,2],
  [2,2]
]
输出: False
解释: 
对角线"[1, 2]"上的元素不同。
```

说明:

- matrix 是一个包含整数的二维数组。
- matrix 的行数和列数均在 [1, 20]范围内。
- matrix[i][j] 包含的整数在 [0, 99]范围内。

进阶:

1. 如果矩阵存储在磁盘上，并且磁盘内存是有限的，因此一次最多只能将一行矩阵加载到内存中，该怎么办？
2. 如果矩阵太大以至于只能一次将部分行加载到内存中，该怎么办？

方法一：对角线法

首先要想明白的是怎么判断 (r1, c1 和 (r2, c2) 这两个点属于一条对角线。通过观察可以发现，在满足 r1 - c1 == r2 - c2 的情况下，这两个点属于同一条对角线。

`groups[r-c]` 存储每条对角线上遇到的第一个元素的值，如果之后遇到的任何一个值不等于之前存储的值，那么这个矩阵就不是托普利茨矩阵，否则就是。

```java
class Solution {
    public boolean isToeplitzMatrix(int[][] matrix) {
        Map<Integer, Integer> groups = new HashMap();
        for (int r = 0; r < matrix.length; ++r) {
            for (int c = 0; c < matrix[0].length; ++c) {
                if (!groups.containsKey(r-c))
                    groups.put(r-c, matrix[r][c]);
                else if (groups.get(r-c) != matrix[r][c])
                    return false;
            }
        }
        return true;
    }
}
```

- 时间复杂度: O(M*N)，即矩阵大小
- 空间复杂度: O(M+N)

方法二： 检查左上邻居

```java
class Solution {
    public boolean isToeplitzMatrix(int[][] matrix) {
        for (int r = 0; r < matrix.length; ++r)
            for (int c = 0; c < matrix[0].length; ++c)
                if (r > 0 && c > 0 && matrix[r-1][c-1] != matrix[r][c])
                    return false;
        return true;
    }
}
```

- 时间复杂度: O(M*N)
- 空间复杂度: O(1)

### [0142] 三维形体投影面积

在 N * N 的网格中，我们放置了一些与 x，y，z 三轴对齐的 1 * 1 * 1 立方体。

每个值 v = grid[i][j] 表示 v 个正方体叠放在单元格 (i, j) 上。

现在，我们查看这些立方体在 xy、yz 和 zx 平面上的投影。

投影就像影子，将三维形体映射到一个二维平面上。

在这里，从顶部、前面和侧面看立方体时，我们会看到“影子”。

返回所有三个投影的总面积。

 示例 1：

```
输入：[[2]]
输出：5
```

示例 2：

```
输入：[[1,2],[3,4]]
输出：17
解释：
这里有该形体在三个轴对齐平面上的三个投影(“阴影部分”)。
```

![img](https://s3-lc-upload.s3.amazonaws.com/uploads/2018/08/02/shadow.png)

示例 3：

```
输入：[[1,0],[0,2]]
输出：8
```

示例 4：

```
输入：[[1,1,1],[1,0,1],[1,1,1]]
输出：14
```


示例 5：

```
输入：[[2,2,2],[2,1,2],[2,2,2]]
输出：21
```


提示：

- 1 <= grid.length = grid[0].length <= 50
- 0 <= grid[i][j] <= 50

方法一：数学

从顶部看，由该形状生成的阴影将是网格中非零值的数目。

从侧面看，由该形状生成的阴影将是网格中每一行的最大值。

从前面看，由该形状生成的阴影将是网格中每一列的最大值。

```java
class Solution {
    public int projectionArea(int[][] grid) {
        int N = grid.length;
        int ans = 0;

        for (int i = 0; i < N;  ++i) {
            int bestRow = 0;  // largest of grid[i][j]
            int bestCol = 0;  // largest of grid[j][i]
            for (int j = 0; j < N; ++j) {
                if (grid[i][j] > 0) ans++;  // top shadow
                bestRow = Math.max(bestRow, grid[i][j]);
                bestCol = Math.max(bestCol, grid[j][i]);
            }
            ans += bestRow + bestCol;
        }

        return ans;
    }
}
```

- 时间复杂度：O(N^2)，其中 N 是 `grid` 的长度
- 空间复杂度：O(1)

### [0143] 删除回文子序列

给你一个字符串 s，它仅由字母 'a' 和 'b' 组成。每一次删除操作都可以从 s 中删除一个回文 子序列。

返回删除给定字符串中所有字符（字符串为空）的最小删除次数。

「子序列」定义：如果一个字符串可以通过删除原字符串某些字符而不改变原字符顺序得到，那么这个字符串就是原字符串的一个子序列。

「回文」定义：如果一个字符串向后和向前读是一致的，那么这个字符串就是一个回文。

 示例 1：

```
输入：s = "ababa"
输出：1
解释：字符串本身就是回文序列，只需要删除一次。
```

示例 2：

```
输入：s = "abb"
输出：2
解释："abb" -> "bb" -> "". 
先删除回文子序列 "a"，然后再删除 "bb"。
```


示例 3：

```
输入：s = "baabb"
输出：2
解释："baabb" -> "b" -> "". 
先删除回文子序列 "baab"，然后再删除 "b"。
```

示例 4：

```
输入：s = ""
输出：0
```


提示：

```
0 <= s.length <= 1000
s 仅包含字母 'a'  和 'b'
```

方法一：

- 原始字符串本身为空，次数就是 0
- 原始字符串本身就是一个回文字符串，那么我们需要的次数就是 1
- 其他字符串就是需要2次即可

```java
class Solution {
    public int removePalindromeSub(String s) {
        if ("".equals(s)) return 0;
        if (s.equals(new StringBuilder(s).reverse().toString())) return 1;
        return 2;
    }
}
```

### [0144] 判定是否互为字符重排

给定两个字符串 s1 和 s2，请编写一个程序，确定其中一个字符串的字符重新排列后，能否变成另一个字符串。

示例 1：

```
输入: s1 = "abc", s2 = "bca"
输出: true 
```

示例 2：

```
输入: s1 = "abc", s2 = "bad"
输出: false
```

说明：

```
0 <= len(s1) <= 100
0 <= len(s2) <= 100
```

方法一：

```java
class Solution {
    public boolean CheckPermutation(String s1, String s2) {
        if(s1.length()!=s2.length()) return false;
        int[] nums = new int[26];
        int len = s1.length();
        for (int i = 0; i < len; i++) {
            nums[s1.charAt(i)-97]++;
            nums[s2.charAt(i)-97]--;
        }
        for (int num : nums) {
            if (num != 0) return false;
        }
        return true;
    }
}
```

