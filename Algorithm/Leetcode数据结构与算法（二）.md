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

### [0021]返回倒数第 k 个节点

实现一种算法，找出单向链表中倒数第 k 个节点。返回该节点的值。

注意：本题相对原题稍作改动

示例：

```
输入： 1->2->3->4->5 和 k = 2
输出： 4
```


说明：

给定的 k 保证是有效的。

方法一：双指针

```java
class Solution {
    public int kthToLast(ListNode head, int k) {

        ListNode temp = head;

        while(k-- != 0){
            temp = temp.next;
        }

        while(temp != null){
            temp = temp.next;
            head = head.next;
        }

        return head.val;
    }
}
```

方法二：递归

 ```java
class Solution {
    // 开始全局变量 K 保持不变
    int pos = 0;
    public int kthToLast(ListNode head, int k) {
        // 当节点在最末尾时触发返回
        if (head == null) return 0;
        // 返回的值
        int val = kthToLast(head.next, k);
        pos++;
        if (pos == k) {
            return head.val;
        }
        return val;
    }
}
 ```

### [0022]最小高度树

给定一个有序整数数组，元素各不相同且按升序排列，编写一个算法，创建一棵高度最小的二叉搜索树。

示例:

给定有序数组: [-10,-3,0,5,9],

一个可能的答案是：[0,-3,9,-10,null,5]，它可以表示下面这个高度平衡二叉搜索树：

          0 
         / \ 
       -3   9 
       /   / 
     -10  5 

 方法：

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
    public TreeNode sortedArrayToBST(int[] nums) {
        if(nums.length == 0) return null;
        TreeNode n = new TreeNode(nums[nums.length/2]);
        n.left = sortedArrayToBST(Arrays.copyOfRange(nums,0,nums.length/2));
        n.right = sortedArrayToBST(Arrays.copyOfRange(nums,nums.length/2+1,nums.length));
        return n;
    }
}
```

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
    public TreeNode sortedArrayToBST(int[] nums) {
        return helper(nums,0,nums.length-1);
    }
    private TreeNode helper(int[] nums,int left,int right){
        if (left>right)
            return null;
        int mid=(left+right)/2;	//(left+right+1)/2;
        TreeNode node=new TreeNode(nums[mid]);
        node.left=helper(nums,left,mid-1);
        node.right=helper(nums,mid+1,right);
        return node;
    }
}
```

###[0023]链表中倒数第k个节点(同0021)

输入一个链表，输出该链表中倒数第k个节点。为了符合大多数人的习惯，本题从1开始计数，即链表的尾节点是倒数第1个节点。例如，一个链表有6个节点，从头节点开始，它们的值依次是1、2、3、4、5、6。这个链表的倒数第3个节点是值为4的节点。

示例：

```
给定一个链表: 1->2->3->4->5, 和 k = 2.

返回链表 4->5.
```

方法一

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
    public ListNode getKthFromEnd(ListNode head, int k) {
        ListNode temp = head;
        while(k-- != 0){
            temp = temp.next;
        }
        while(temp != null){
            temp = temp.next;
            head = head.next;
        }
        return head;
    }
}
```

### [0024]二叉树的深度

输入一棵二叉树的根节点，求该树的深度。从根节点到叶节点依次经过的节点（含根、叶节点）形成树的一条路径，最长路径的长度为树的深度。

例如：

给定二叉树 `[3,9,20,null,null,15,7]`，

```
    3
   / \
  9  20
    /  \
   15   7
```

返回它的最大深度 3 。

提示：

1. `节点总数 <= 10000`

方法一：递归

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
    
    public int maxDepth(TreeNode root) {
       return root==null ? 0:Math.max(maxDepth(root.left),maxDepth(root.right))+1;
    }
}
```

方法二：栈

本质上是后序遍历

非递归可以利用栈。

把根节点root先压入栈，创建两个TreeNode节点，一个用于返回实时的栈顶元素，一个用于返回先前已访问过节点。对栈判空+循环，开始遍历，【若当前访问的节点为叶子即无子树，则弹出当前节点并返回至上一个节点并访问其另外的子树检查是否访问过，若没有则遍历一直到叶子为止】，重复这个操作，即可遍历整棵树得到深度

```java
class Solution {
    
    public int maxDepth(TreeNode root) {
       
        if(root == null) return 0;
        Stack<TreeNode> stack = new Stack<>();
        stack.push(root);
        TreeNode top; //一个用于返回实时的栈顶元素
        TreeNode lastVisit = root;//一个用于返回先前已访问过节点
        int cnt = 0;
        while(!stack.isEmpty()){
            cnt = Math.max(cnt,stack.size());
            top = stack.peek();
            if(top.left != null && lastVisit != top.left && lastVisit != top.right){
                stack.push(top.left);
            }
            else if(top.right != null && lastVisit != top.right){
                stack.push(top.right);
            }
            else{
                lastVisit = stack.pop();
            }
            
        }

        return cnt;
    }
}
```

方法3：双向队列Deque

本质上是层次遍历，也可视为root数组的从左到右的遍历

非递归还能利用双向队列

把根节点放入队列后再取出接着对其左右子树判空，有则加入分别加入队尾，再依次对剩余节点子树进行判空操作，若为叶子节点则使得当前节点指向队尾（下一次循环则下一层），此时当前层次遍历完毕，深度+1。

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
    
    public int maxDepth(TreeNode root) {
       
        if(root == null) return 0;
        
        Deque<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        TreeNode queueLast = root;
        TreeNode nowVisit;
        int depth = 0;
        
        while(!queue.isEmpty()){
            nowVisit = queue.poll();
            if(nowVisit.left != null) queue.offer(nowVisit.left);
            if(nowVisit.right != null) queue.offer(nowVisit.right);
            if(nowVisit == queueLast){
                queueLast = queue.peekLast();
                depth++;
            }
        }
        return depth;
    }
}
```

###[0025]二叉树的镜像  

请完成一个函数，输入一个二叉树，该函数输出它的镜像。

例如输入：

```
     4
   /   \
  2     7
 / \   / \
1   3 6   9
```

镜像输出：

```
     4
   /   \
  7     2
 / \   / \
9   6 3   1

```

**示例 1：**

```
输入：root = [4,2,7,1,3,6,9]
输出：[4,7,2,9,6,3,1]
```

**限制：**

```
0 <= 节点个数 <= 1000
```

方法一：递归

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
    TreeNode temp = null;
    public TreeNode mirrorTree(TreeNode root) {        
        if(root == null){
            return root;
        }
        temp = root.left;
        root.left = root.right;
        root.right = temp;
        mirrorTree(root.left);
        mirrorTree(root.right);
        return root;
    }

} 
```

时间复杂度：每个元素都必须访问一次，所以是O(n)
空间复杂度：最坏的情况下，需要存放O(h)个函数调用(h是树的高度)，所以是O(h)

方法二：迭代

递归实现也就是深度优先遍历的方式，那么对应的就是广度优先遍历。
广度优先遍历需要额外的数据结构--队列，来存放临时遍历到的元素。
深度优先遍历的特点是一竿子插到底，不行了再退回来继续；而广度优先遍历的特点是层层扫荡。
所以，我们需要先将根节点放入到队列中，然后不断的迭代队列中的元素。
对当前元素调换其左右子树的位置，然后：

- 判断其左子树是否为空，不为空就放入队列中
- 判断其右子树是否为空，不为空就放入队列中

动态图如下：

![226_迭代.gif](https://pic.leetcode-cn.com/f9e06159617cbf8372b544daee37be70286c3d9b762c016664e225044fc4d479-226_%E8%BF%AD%E4%BB%A3.gif)

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
    public TreeNode mirrorTree(TreeNode root) {        
        if(root==null) {
			return null;
		}
		//将二叉树中的节点逐层放入队列中，再迭代处理队列中的元素
		LinkedList<TreeNode> queue = new LinkedList<TreeNode>();
		queue.add(root);
		while(!queue.isEmpty()) {
			//每次都从队列中拿一个节点，并交换这个节点的左右子树
			TreeNode tmp = queue.poll();
			TreeNode left = tmp.left;
			tmp.left = tmp.right;
			tmp.right = left;
			//如果当前节点的左子树不为空，则放入队列等待后续处理
			if(tmp.left!=null) {
				queue.add(tmp.left);
			}
			//如果当前节点的右子树不为空，则放入队列等待后续处理
			if(tmp.right!=null) {
				queue.add(tmp.right);
			}
			
		}
		//返回处理完的根节点
		return root;
    }

}
```

###[0026]打印从1到最大的n位数

输入数字 `n`，按顺序打印出从 1 到最大的 n 位十进制数。比如输入 3，则打印出 1、2、3 一直到最大的 3 位数 999。

**示例 1:**

```
输入: n = 1
输出: [1,2,3,4,5,6,7,8,9]
```

说明：

- 用返回一个整数列表来代替打印
- n 为正整数

方法一：

```java
class Solution {
    public int[] printNumbers(int n) {
        int[] r = new int[(int)Math.pow(10,n) -1];
        for(int i = 0; i < r.length;i++){
            r[i] = i+1;
        }
        return r;
    }
}
```

方法二：

```java
class Solution {
    StringBuilder sb;
    int idx = 0;
    public boolean increment(int n){
        boolean carry=false;
        for(int i=0;i<sb.length();++i){
            if(carry || i==0){
                if(sb.charAt(i)=='9'){
                    sb.setCharAt(i,'0');
                    carry = true;
                }else{
                    sb.setCharAt(i,(char) (sb.charAt(i)+1));
                    carry = false;
                }
            }else{
                break; // no addition on last idx, no need to compute any more
            }
        }
        if(carry){
            sb.append("1");
        }
        return sb.length()<=n; // overflow!
    }

    public void save(int ans[]){
        ans[idx] = Integer.parseInt(sb.reverse().toString());
        sb.reverse();
    }

    public int[] printNumbers(int n) {
        int[] ans = new int[(int) Math.pow(10,n) - 1];
        sb = new StringBuilder("0");
        while(increment(n)){
            save(ans);
            idx++;
        }
        return ans;
    }
}
```

**拓展**

- 大数打印：可以设定一个阈值，例如`long`的最大值，当超过了这个最大值，将阈值转为字符数组`toCharArray()`，然后继续`+1`打印，这样可以提高时间效率，因为一部分的数仍是`O(1)`打印
- 全排列：求从`1~pow(10,n)-1`实际上可以转化为`0-9`在`n`个位置上的全排列

### [0027]替换空格

请实现一个函数，把字符串 s 中的每个空格替换成"%20"。 

示例 1：

```
输入：s = "We are happy."
输出："We%20are%20happy."
```


限制：

0 <= s 的长度 <= 10000

方法一：

```java
class Solution {
    public String replaceSpace(String s) {
        return s.replace(" ","%20");
    }
}
```

方法二：

```java

class Solution {
    public String replaceSpace(String s) {
        int length = s.length();
        char[] array = new char[length * 3];
        int size = 0;
        for (int i = 0; i < length; i++) {
            char c = s.charAt(i);
            if (c == ' ') {
                array[size++] = '%';
                array[size++] = '2';
                array[size++] = '0';
            } else {
                array[size++] = c;
            }
        }
        String newStr = new String(array, 0, size);
        return newStr;
    }
}

```

###[0028]删除中间节点

实现一种算法，删除单向链表中间的某个节点（除了第一个和最后一个节点，不一定是中间节点），假定你只能访问该节点。

示例：

```
输入：单向链表a->b->c->d->e->f中的节点c
结果：不返回任何数据，但该链表变为a->b->d->e->f
```

方法：

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

### [0029]分割平衡字符串

在一个「平衡字符串」中，'L' 和 'R' 字符的数量是相同的。

给出一个平衡字符串 s，请你将它分割成尽可能多的平衡字符串。

返回可以通过分割得到的平衡字符串的最大数量。



示例 1：

```
输入：s = "RLRRLLRLRL"
输出：4
解释：s 可以分割为 "RL", "RRLL", "RL", "RL", 每个子字符串中都包含相同数量的 'L' 和 'R'。
```

示例 2：

```
输入：s = "RLLLLRRRLR"
输出：3
解释：s 可以分割为 "RL", "LLLRRR", "LR", 每个子字符串中都包含相同数量的 'L' 和 'R'。
```


示例 3：

```
输入：s = "LLLLRRRR"
输出：1
解释：s 只能保持原样 "LLLLRRRR".
```

提示：

```
1 <= s.length <= 1000
s[i] = 'L' 或 'R'
```



方法一：数组模拟栈

可以利用栈的FILO的特性解题。核心算法是通过取容器顶部的字母来甄别是否与未来要放入容器的字母是互异的。

每次遇到不一样的，我们把当前字母擦除掉（通过指针位置移动就行）

遇到一样的，就堆积在容器中。

但每次擦除完，都检查容器的 size 是否等于 0，是则表明堆积的都被消灭了，找到了一个平衡字符串，计数器加 1。

```java
class Solution {
    public int balancedStringSplit(String s) {
        int count = 0;
        int[] container = new int[s.length()];
        int size = 0;

        for (char c : s.toCharArray()) {
            if (size == 0 || c == container[size-1]) 
                container[size++] = c;              //放入容器
            else if (c != container[size-1]){
                size--;                             //匹配到了互异字母
                if (size == 0)
                    count++;
            }
        }
        return count;
    }
}
```

时间复杂度：O(N)，N 为字符串 s 的长度。

空间复杂度：O(N)，使用了大小为N 的数组作为容器。

方法二：计数法

首先以 R 为基准，设立一个 专门记录 R 的数量的计数器 counter_R。

- 遍历字符串 s，是 'R' 计数器 couter_R 加 1

- 遇到 'L' 则认为是消灭 R 的，counter_R 自减 1

- 每次消灭 R，都遍历完一次，检查在这之前的 R 是否还有剩余

- 没有剩余的话，该位置之前的算是一个由相同数量 L 与 R 组成的 「平衡字符串」

```java
public int balancedStringSplit1(String s) {
    int count = 0;
    int counter_R = 0;
    for (int i = 0; i < s.length(); i++) {
        if (s.charAt(i) == 'R')
            counter_R++;    //记录R
        else 
            counter_R--;    //消灭R
        if (counter_R == 0)
            count++;	    //证明在i位置之前的字符都有相同数量的L和R
    }
    return count;
}
```

- 时间复杂度：O(N)，N为字符串 s的长度。
- 空间复杂度：O(1)，只使用常数级别的空间。

### [0030] 删除最外层的括号

有效括号字符串为空 ("")、"(" + A + ")" 或 A + B，其中 A 和 B 都是有效的括号字符串，+ 代表字符串的连接。例如，""，"()"，"(())()" 和 "(()(()))" 都是有效的括号字符串。

如果有效字符串 S 非空，且不存在将其拆分为 S = A+B 的方法，我们称其为原语（primitive），其中 A 和 B 都是非空有效括号字符串。

给出一个非空有效字符串 S，考虑将其进行原语化分解，使得：S = P_1 + P_2 + ... + P_k，其中 P_i 是有效括号字符串原语。

对 S 进行原语化分解，删除分解中每个原语字符串的最外层括号，返回 S 。

示例 1：

```
输入："(()())(())"
输出："()()()"
解释：
输入字符串为 "(()())(())"，原语化分解得到 "(()())" + "(())"，
删除每个部分中的最外层括号后得到 "()()" + "()" = "()()()"。
```


示例 2：

```
输入："(()())(())(()(()))"
输出："()()()()(())"
解释：
输入字符串为 "(()())(())(()(()))"，原语化分解得到 "(()())" + "(())" + "(()(()))"，
删除每个部分中的最外层括号后得到 "()()" + "()" + "()(())" = "()()()()(())"。
```


示例 3：

```
输入："()()"
输出：""
解释：
输入字符串为 "()()"，原语化分解得到 "()" + "()"，
删除每个部分中的最外层括号后得到 "" + "" = ""。
```

提示：

```
S.length <= 10000
S[i] 为 "(" 或 ")"
S 是一个有效括号字符串
```

方法：

```java
class Solution {
    public String removeOuterParentheses(String S) {
        StringBuilder sb = new StringBuilder();
        int level = 0;
        for (char c : S.toCharArray()) {
            if (c == ')') --level;
            if (level >= 1) sb.append(c);
            if (c == '(') ++level;
        }
        return sb.toString();
    }
}
```

### [0031]按既定顺序创建目标数组

给你两个整数数组 nums 和 index。你需要按照以下规则创建目标数组：

目标数组 target 最初为空。
按从左到右的顺序依次读取 nums[i] 和 index[i]，在 target 数组中的下标 index[i] 处插入值 nums[i] 。
重复上一步，直到在 nums 和 index 中都没有要读取的元素。
请你返回目标数组。

题目保证数字插入位置总是存在。

示例 1：

```
输入：nums = [0,1,2,3,4], index = [0,1,2,2,1]
输出：[0,4,1,3,2]
解释：
nums       index     target
0            0        [0]
1            1        [0,1]
2            2        [0,1,2]
3            2        [0,1,3,2]
4            1        [0,4,1,3,2]
```


示例 2：

```
输入：nums = [1,2,3,4,0], index = [0,1,2,3,0]
输出：[0,1,2,3,4]
解释：
nums       index     target
1            0        [1]
2            1        [1,2]
3            2        [1,2,3]
4            3        [1,2,3,4]
0            0        [0,1,2,3,4]
```

示例 3：

```
输入：nums = [1], index = [0]
输出：[1]
```


提示：

```
1 <= nums.length, index.length <= 100
nums.length == index.length
0 <= nums[i] <= 100
0 <= index[i] <= i
```

  方法一：List

```java
class Solution {
    public int[] createTargetArray(int[] nums, int[] index) {
        List<Integer> target = new ArrayList();
        for(int i=0; i<index.length; i++){
            target.add(index[i], nums[i]);
        }
        int[] ret = new int[target.size()];
        for(int i = 0; i < target.size(); i++){
            ret[i] = target.get(i);
        }
        return ret;
    }
}
```

 方法二：数组

```java
class Solution {
    public int[] createTargetArray(int[] nums, int[] index) {
       int[] target = new int[nums.length];
       int pos = -1;
       for(int i=0; i<index.length; i++){
           pos++;
  
        for(int j = pos; j > index[i] ; j--){
            target[j] = target[j-1];
        }
                  
        target[index[i]] = nums[i];
        
       }
       return target;
    }
}
```

### [0032]二叉搜索树的范围和



给定二叉搜索树的根结点 root，返回 L 和 R（含）之间的所有结点的值的和。

二叉搜索树保证具有唯一的值。

示例 1：

```
输入：root = [10,5,15,3,7,null,18], L = 7, R = 15
输出：32
```


示例 2：

```
输入：root = [10,5,15,3,7,13,18,1,null,6], L = 6, R = 10
输出：23
```

提示：

```
树中的结点数量最多为 10000 个。
最终的答案保证小于 2^31。
```

 方法一：

```java
class Solution {
    public int rangeSumBST(TreeNode root, int L, int R) {
        if (root == null) {
            return 0;
        }
        if (root.val < L) {
            return rangeSumBST(root.right, L, R);
        }
        if (root.val > R) {
            return rangeSumBST(root.left, L, R);
        }
        return root.val + rangeSumBST(root.left, L, R) + rangeSumBST(root.right, L, R);
    }
}
```

方法二：

```java
class Solution {
    public int rangeSumBST(TreeNode root, int L, int R) {
        int ans = 0;
        Stack<TreeNode> stack = new Stack();
        stack.push(root);
        while (!stack.isEmpty()) {
            TreeNode node = stack.pop();
            if (node != null) {
                if (L <= node.val && node.val <= R)
                    ans += node.val;
                if (L < node.val)
                    stack.push(node.left);
                if (node.val < R)
                    stack.push(node.right);
            }
        }
        return ans;
    }
}
```






