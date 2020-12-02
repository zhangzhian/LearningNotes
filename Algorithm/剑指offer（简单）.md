# 剑指offer：简单部分

### [001] 左旋转字符串

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

- 1 <= k < s.length <= 10000

方法一：字符串切片

```java
class Solution {
    public String reverseLeftWords(String s, int n) {
        return s.substring(n, s.length()) + s.substring(0, n);
    }
}
```

时间复杂度 O(N)

空间复杂度 O(N)

方法二：列表遍历拼接

```java
class Solution {
    public String reverseLeftWords(String s, int n) {
        StringBuilder res = new StringBuilder();
        for(int i = n; i < s.length(); i++)
            res.append(s.charAt(i));
        for(int i = 0; i < n; i++)
            res.append(s.charAt(i));
        return res.toString();
    }
}
```

```java
class Solution {
    public String reverseLeftWords(String s, int n) {
        StringBuilder res = new StringBuilder();
        for(int i = n; i < n + s.length(); i++)
            //利用求余运算，可以简化代码。
            res.append(s.charAt(i % s.length()));
        return res.toString();
    }
}
```

时间复杂度 O(N)

空间复杂度 O(N)

方法三：字符串遍历拼接

```java
class Solution {
    public String reverseLeftWords(String s, int n) {
        String res = "";
        for(int i = n; i < s.length(); i++)
            res += s.charAt(i);
        for(int i = 0; i < n; i++)
            res += s.charAt(i);
        return res;
    }
}
```

```java
class Solution {
    public String reverseLeftWords(String s, int n) {
        String res = "";
        for(int i = n; i < n + s.length(); i++)
            res += s.charAt(i % s.length());
        return res;
    }
}
```

![Picture4.png](https://pic.leetcode-cn.com/ef68413b3366b97af3ed76037c6a9d1e40ac09c74fd6e5cb6d5173cbd7116beb-Picture4.png)

### [002]   二叉树的深度

输入一棵二叉树的根节点，求该树的深度。从根节点到叶节点依次经过的节点（含根、叶节点）形成树的一条路径，最长路径的长度为树的深度。

例如：

给定二叉树 [3,9,20,null,null,15,7]，

    	3
       / \
      9  20
        /  \
       15   7
    返回它的最大深度 3 。
方法一：递归，深度优先

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
        if (root == null) {
            return 0;
        } else {
            int leftHeight = maxDepth(root.left);
            int rightHeight = maxDepth(root.right);
            return Math.max(leftHeight, rightHeight) + 1;
        }
    }
}
```

时间复杂度：O(n)

空间复杂度：O(H)，其中 H 表示二叉树的高度。

方法二：广度优先

```java
class Solution {
    public int maxDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }
        Queue<TreeNode> queue = new LinkedList<TreeNode>();
        queue.offer(root);//添加失败返回false。add抛出异常
        int ans = 0;
        while (!queue.isEmpty()) {
            int size = queue.size();//记录该层多少个节点
            while (size > 0) {
                TreeNode node = queue.poll();//失败返回null，romove异常
                if (node.left != null) {
                    queue.offer(node.left);
                }
                if (node.right != null) {
                    queue.offer(node.right);
                }
                size--;
            }
            ans++;//层数加1
        }
        return ans;
    }
}
```

时间复杂度：O(n)

空间复杂度：最坏情况下会达到 O(n)

### [003] 二叉树的镜像(翻转二叉树)

请完成一个函数，输入一个二叉树，该函数输出它的镜像。

例如输入：

     	  4
        /   \
      2     7
     / \   / \
    1   3 6   9

镜像输出：

     	  4
        /   \
      7     2
     / \   / \
    9   6 3   1
示例 1：

```
输入：root = [4,2,7,1,3,6,9]
输出：[4,7,2,9,6,3,1]
```


限制：

- 0 <= 节点个数 <= 1000

方法一：递归-从下到上

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
    public TreeNode invertTree(TreeNode root) {
        if (root == null) {
            return null;
        }
        TreeNode left = invertTree(root.left);
        TreeNode right = invertTree(root.right);
        root.left = right;
        root.right = left;
        return root;
    }
}
```

递归-从上到下

```java
class Solution {
	public TreeNode invertTree(TreeNode root) {
		//递归函数的终止条件，节点为空时返回
		if(root==null) {
			return null;
		}
		//下面三句是将当前节点的左右子树交换
		TreeNode tmp = root.right;
		root.right = root.left;
		root.left = tmp;
		//递归交换当前节点的 左子树
		invertTree(root.left);
		//递归交换当前节点的 右子树
		invertTree(root.right);
		//函数返回时就表示当前这个节点，以及它的左右子树
		//都已经交换完了
		return root;
	}
}
```

时间复杂度：O(N)

空间复杂度：O(N)

方法二：

```java
class Solution {
	public TreeNode invertTree(TreeNode root) {
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

时间复杂度：O(N)

空间复杂度：O(N)

### [004] 链表中倒数第k个节点

输入一个链表，输出该链表中倒数第k个节点。为了符合大多数人的习惯，本题从1开始计数，即链表的尾节点是倒数第1个节点。例如，一个链表有6个节点，从头节点开始，它们的值依次是1、2、3、4、5、6。这个链表的倒数第3个节点是值为4的节点。

 示例：

```
给定一个链表: 1->2->3->4->5, 和 k = 2.

返回链表 4->5.
```

方法一：双指针

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
        ListNode pre = head;
        int pos = 0;
        while(head != null){       
           head = head.next;
           pos++;
           if(pos > k){
               pre = pre.next;
           }       
        }
        return pre;
    }
}
```

时间复杂度：O(N)

空间复杂度：O(1)

方法二：双指针，不需要计数

```java
class Solution {
    public ListNode getKthFromEnd(ListNode head, int k) {
        ListNode former = head, latter = head;
        for(int i = 0; i < k; i++)
            former = former.next;
        while(former != null) {
            former = former.next;
            latter = latter.next;
        }
        return latter;
    }
}
```

时间复杂度：O(N)

空间复杂度：O(1)

方法三：反转2次

```java

class Solution {
    public ListNode getKthFromEnd(ListNode head, int k) {
        ListNode p = reverse(head);
        ListNode keepreverse = p;
        while(k>1){
            p = p.next;
            k--;
        }
        p.next = null;
        return reverse(keepreverse);
    }
    public ListNode reverse(ListNode s){
        ListNode pre = s;
        ListNode now = s.next;
        ListNode temp;
        pre.next = null;
        while(now!=null){
            temp = now.next;
            now.next = pre;
            pre = now;
            now = temp;
        }
        return pre;
    }
}
```

方法四：栈

```java
class Solution {
    public ListNode getKthFromEnd(ListNode head, int k) {
        Stack<ListNode> stack = new Stack<>();
        //链表节点压栈
        while (head != null) {
            stack.push(head);
            head = head.next;
        }
        //在出栈串成新的链表
        ListNode ret = stack.pop();
        while (--k > 0) {
            ret = stack.pop();
        }
        return ret;
    }
}
```

方法五：递归

```java
class Solution {
    int size;
    public ListNode getKthFromEnd(ListNode head, int k) {
        if (head == null)
            return head;
        ListNode node = getKthFromEnd(head.next, k);
        if (++size == k)
            return head;
        return node;
    }
}
```

### [005※] 打印从1到最大的n位数

输入数字 n，按顺序打印出从 1 到最大的 n 位十进制数。比如输入 3，则打印出 1、2、3 一直到最大的 3 位数 999。

示例 1:

```
输入: n = 1
输出: [1,2,3,4,5,6,7,8,9]
```


说明：

- 用返回一个整数列表来代替打印
- n 为正整数

方法一：直接计算

```java
class Solution {
    public int[] printNumbers(int n) {
        int end = (int)Math.pow(10, n) - 1;
        int[] res = new int[end];
        for(int i = 0; i < end; i++)
            res[i] = i + 1;
        return res;
    }
}
```

时间复杂度 O(10^n)

空间复杂度 O(1)

**方法二：[大数打印解法](https://leetcode-cn.com/problems/da-yin-cong-1dao-zui-da-de-nwei-shu-lcof/solution/mian-shi-ti-17-da-yin-cong-1-dao-zui-da-de-n-wei-2/)**

实际上，本题的主要考点是大数越界情况下的打印。需要解决以下三个问题：

1. 表示大数的变量类型：无论是 short / int / long ... 任意变量类型，数字的取值范围都是有限的。大数的表示应用字符串 String 类型。
2. 生成数字的字符串集：生成的列表实际上是 n 位 00 - 99 的 全排列 ，因此可避开进位操作，通过递归生成数字的 String 列表。

3. 递归生成全排列：
基于分治算法的思想，先固定高位，向低位递归，当个位已被固定时，添加数字的字符串。例如当 n = 2 时（数字范围 1 - 99），固定十位为 00 - 99 ，按顺序依次开启递归，固定个位 00 - 99 ，终止递归并添加数字字符串。

![Picture1.png](https://pic.leetcode-cn.com/83f4b5930ddc1d42b05c724ea2950ee7f00427b11150c86b45bd88405f8c7c87-Picture1.png)

为 **正确表示大数** ，以下代码的返回值为数字字符串集拼接而成的长字符串。

```java
class Solution {
    StringBuilder res;
    int nine = 0, count = 0, start, n;
    char[] num, loop = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9'};
    public String printNumbers(int n) {
        this.n = n;
        res = new StringBuilder();
        num = new char[n];
        start = n - 1;
        dfs(0);
        res.deleteCharAt(res.length() - 1);
        return res.toString();
    }
    void dfs(int x) {
        if(x == n) {
            String s = String.valueOf(num).substring(start);
            if(!s.equals("0")) res.append(s + ",");
            if(n - start == nine) start--;
            return;
        }
        for(char i : loop) {
            if(i == '9') nine++;
            num[x] = i;
            dfs(x + 1);
        }
        nine--;
    }
}
```

本题要求输出 int 类型数组。为 **运行通过** ，可在添加数字字符串 s*s* 前，将其转化为 int 类型。代码如下所示：

```java
class Solution {
    int[] res;
    int nine = 0, count = 0, start, n;
    char[] num, loop = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9'};
    public int[] printNumbers(int n) {
        this.n = n;
        res = new int[(int)Math.pow(10, n) - 1];
        num = new char[n];
        start = n - 1;
        dfs(0);
        return res;
    }
    void dfs(int x) {
        if(x == n) {
            String s = String.valueOf(num).substring(start);
            if(!s.equals("0")) res[count++] = Integer.parseInt(s);
            if(n - start == nine) start--;
            return;
        }
        for(char i : loop) {
            if(i == '9') nine++;
            num[x] = i;
            dfs(x + 1);
        }
        nine--;
    }
}
```

时间复杂度 O(10^n)

空间复杂度 O(10^n)

### [006] 替换空格

请实现一个函数，把字符串 s 中的每个空格替换成"%20"。

 示例 1：

```
输入：s = "We are happy."
输出："We%20are%20happy."
```

**限制：**

- 0 <= s 的长度 <= 10000

方法一：字符数组

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

- 时间复杂度：O(n) 
- 空间复杂度：O(n) 

方法二：

```java
class Solution {
    public String replaceSpace(String s) {
        StringBuilder res = new StringBuilder();
        for(Character c : s.toCharArray())
        {
            if(c == ' ') res.append("%20");
            else res.append(c);
        }
        return res.toString();
    }
}
```

- 时间复杂度：O(n) 
- 空间复杂度：O(n) 

### [007] 从尾到头打印链表

输入一个链表的头节点，从尾到头反过来返回每个节点的值（用数组返回）。

 示例 1：

```
输入：head = [1,3,2]
输出：[2,3,1]
```


限制：

- 0 <= 链表长度 <= 10000

方法一：递归

```java
class Solution {
    ArrayList<Integer> tmp = new ArrayList<Integer>();
    public int[] reversePrint(ListNode head) {
        recur(head);
        int[] res = new int[tmp.size()];
        for(int i = 0; i < res.length; i++)
            res[i] = tmp.get(i);
        return res;
    }
    void recur(ListNode head) {
        if(head == null) return;
        recur(head.next);
        tmp.add(head.val);
    }
}
```

不需要重新遍历ArrayList

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
    int[] res;
    int i = 0, j = 0;
    public int[] reversePrint(ListNode head) {
        solve(head);
        return res;
    }
    public void solve(ListNode head){
        if(head == null){
            res = new int[i];
            return;
        }
        i++; 
        solve(head.next);
        res[j] = head.val;
        j++;
    }
}
```



时间复杂度：O(n)

空间复杂度：O(n)

方法二：辅助栈法

```java
class Solution {
    public int[] reversePrint(ListNode head) {
        LinkedList<Integer> stack = new LinkedList<Integer>();
        while(head != null) {
            stack.addLast(head.val);
            head = head.next;
        }
        int[] res = new int[stack.size()];
        for(int i = 0; i < res.length; i++)
            res[i] = stack.removeLast();
    return res;
    }
}
```

- 时间复杂度：O(n) 
- 空间复杂度：O(n) 

### [008] 反转链表

反转一个单链表。

示例:

```
输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL
```

**限制：**

```
0 <= 节点个数 <= 5000
```

方法一：递归

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
    public ListNode reverseList(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }
        ListNode p = reverseList(head.next);
        head.next.next = head;
        head.next = null;
        return p;
    }
}
```

时间复杂度：O(n)

空间复杂度：O(n)

方法二：

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode prev = null;
        ListNode curr = head;
        while (curr != null) {
            ListNode nextTemp = curr.next;
            curr.next = prev;
            prev = curr;
            curr = nextTemp;
        }
        return prev;
    }
}
```

- 时间复杂度：O(n) 
- 空间复杂度：O(1) 

### [009] 二叉搜索树的第k大节点

给定一棵二叉搜索树，请找出其中第k大的节点。

```
示例 1:

输入: root = [3,1,4,null,2], k = 1
   3
  / \
 1   4
  \
   2
输出: 4
示例 2:

输入: root = [5,3,6,2,4,null,null,1], k = 3
       5
      / \
     3   6
    / \
   2   4
  /
 1
输出: 4
```

**限制：**

- 1 ≤ k ≤ 二叉搜索树元素个数

方法一：dfs

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
    public int kthLargest(TreeNode root, int k) {
        List<Integer> list = new ArrayList();
        dfs(list,root);
        return list.get(list.size() - k);
    }

    void dfs(List list, TreeNode root){
        if (root == null) return;
        dfs(list,root.left);
        list.add(root.val);
        dfs(list,root.right);
    }
}
```

```java
class Solution {
    int res, k;
    public int kthLargest(TreeNode root, int k) {
        this.k = k;
        dfs(root);
        return res;
    }
    void dfs(TreeNode root) {
        if(root == null || k == 0) return;//当root为空或者已经找到了res时，直接返回
        dfs(root.right);
        if(--k == 0) {
            res = root.val;
            return;
        }
        dfs(root.left);
    }
}
```

- 时间复杂度 O(N) 

- 空间复杂度 O(N) 

方法二：

```java
class Solution {
    public int kthLargest(TreeNode root, int k) {
        int count = 1;
        Stack<TreeNode> stack = new Stack<>();
        while (root != null || !stack.empty()) {
            while (root != null) {
                stack.push(root);
                root = root.right;
            }
            TreeNode pop = stack.pop();
            if (count == k) {
                return pop.val;
            }
            count++;
            root = pop.left;
        }
        return 0;
    }
}
```

### [010] 合并两个排序的链表

输入两个递增排序的链表，合并这两个链表并使新链表中的节点仍然是递增排序的。

示例1：

```
输入：1->2->4, 1->3->4
输出：1->1->2->3->4->4
```

限制：

- 0 <= 链表长度 <= 1000

方法一：递归

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if (l1 == null) {
            return l2;
        } else if (l2 == null) {
            return l1;
        } else if (l1.val < l2.val) {
            l1.next = mergeTwoLists(l1.next, l2);
            return l1;
        } else {
            l2.next = mergeTwoLists(l1, l2.next);
            return l2;
        }
    }
}
```

时间复杂度：O(n + m) 

空间复杂度：O(n + m) 

方法二：迭代

```java
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        ListNode prehead = new ListNode(-1);

        ListNode prev = prehead;
        while (l1 != null && l2 != null) {
            if (l1.val <= l2.val) {
                prev.next = l1;
                l1 = l1.next;
            } else {
                prev.next = l2;
                l2 = l2.next;
            }
            prev = prev.next;
        }

        // 合并后 l1 和 l2 最多只有一个还未被合并完，我们直接将链表末尾指向未合并完的链表即可
        prev.next = l1 == null ? l2 : l1;

        return prehead.next;
    }
}
```

时间复杂度：O(n + m)  

空间复杂度：O(1) 

### [011] 二进制中1的个数

请实现一个函数，输入一个整数，输出该数二进制表示中 1 的个数。例如，把 9 表示成二进制是 1001，有 2 位是 1。因此，如果输入 9，则该函数输出 2。

方法一：逐位判断

```java
public class Solution {
    public int hammingWeight(int n) {
        int res = 0;
        while(n != 0) {
            res += n & 1;
            n >>>= 1;
        }
        return res;
    }
}
```

- 时间复杂度 O(log_2 n)其中 log_2 n 代表数字 n 最高位 1 的所在位数

- 空间复杂度 O(1) 

方法二：巧用 n \& (n - 1)n&(n−1)

![Picture10.png](https://pic.leetcode-cn.com/9bc8ab7ba242888d5291770d35ef749ae76ee2f1a51d31d729324755fc4b1b1c-Picture10.png)

```java
public class Solution {
    public int hammingWeight(int n) {
        int res = 0;
        while(n != 0) {
            res++;
            n &= n - 1;
        }
        return res;
    }
}
```

时间复杂度 O(N)，N为1的个数 

空间复杂度 O(1) 

### [012] 用两个栈实现队列

用两个栈实现一个队列。队列的声明如下，请实现它的两个函数 appendTail 和 deleteHead ，分别完成在队列尾部插入整数和在队列头部删除整数的功能。(若队列中没有元素，deleteHead 操作返回 -1 )

示例 1：

```
输入：
["CQueue","appendTail","deleteHead","deleteHead"]
[[],[3],[],[]]
输出：[null,null,3,-1]
```

示例 2：

```
输入：
["CQueue","deleteHead","appendTail","appendTail","deleteHead","deleteHead"]
[[],[],[5],[2],[],[]]
输出：[null,-1,null,null,5,2]
```

提示：

- 1 <= values <= 10000
- 最多会对 appendTail、deleteHead 进行 10000 次调用

方法一：

成员变量

- 维护两个栈 stack1 和 stack2，其中 stack1 支持插入操作，stack2 支持删除操作

构造方法

- 初始化 stack1 和 stack2 为空

插入元素，对应方法 appendTail

- stack1 直接插入元素

删除元素，对应方法 deleteHead

- 如果 stack2 为空，则将 stack1 里的所有元素弹出插入到 stack2 里
- 如果 stack2 仍为空，则返回 -1，否则从 stack2 弹出一个元素并返回

![fig1](https://assets.leetcode-cn.com/solution-static/jianzhi_09/jianzhi_9.gif)

```java
class CQueue {
    Deque<Integer> stack1;
    Deque<Integer> stack2;
    
    public CQueue() {
        stack1 = new LinkedList<Integer>();
        stack2 = new LinkedList<Integer>();
    }
    
    public void appendTail(int value) {
        stack1.push(value);
    }
    
    public int deleteHead() {
        // 如果第二个栈为空
        if (stack2.isEmpty()) {
            while (!stack1.isEmpty()) {
                stack2.push(stack1.pop());
            }
        } 
        if (stack2.isEmpty()) {
            return -1;
        } else {
            int deleteItem = stack2.pop();
            return deleteItem;
        }
    }
}
```

时间复杂度：对于插入和删除操作，时间复杂度均为 O(1)。插入不多说，对于删除操作，虽然看起来是 O(n)的时间复杂度，但是仔细考虑下每个元素只会「至多被插入和弹出 stack2 一次」，因此均摊下来每个元素被删除的时间复杂度仍为 O(1)

空间复杂度：O(n)。需要使用两个栈存储已有的元素。

### [013] 二叉树的最近公共祖先

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

**说明:**

- 所有节点的值都是唯一的。
- p、q 为不同节点且均存在于给定的二叉树中。

方法一：递归

**终止条件：**

- 当越过叶节点，则直接返回 null  ；
- 当 root 等于 p, q，则直接返回 root  ；

**递推工作：**

- 开启递归左子节点，返回值记为 left ；

- 开启递归右子节点，返回值记为 right ；

**返回值**： 根据 left 和 right ，可展开为四种情况；

- 当 left 和 right 同时为空 ：说明 root 的左 / 右子树中都不包含 p,q ，返回 null ；

- 当 left 和 right 同时不为空 ：说明 p, q 分列在 root 的 异侧 （分别在 左 / 右子树），因此 root 为最近公共祖先，返回 root ；

- 当 left 为空 ，right 不为空 ：p,q 都不在 root 的左子树中，直接返回 right 。具体可分为两种情况：
  - p,q 其中一个在 root 的 右子树 中，此时 right 指向 p（假设为 p ）；
  - p,q 两节点都在 root 的 右子树 中，此时的 right 指向 最近公共祖先节点 ；

- 当 left 不为空 ， right 为空 ：与情况 3. 同理；

观察发现， 情况 1. 可合并至 3. 和 4. 内。

```java
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if(root == null || root == p || root == q) return root;
        TreeNode left = lowestCommonAncestor(root.left, p, q);
        TreeNode right = lowestCommonAncestor(root.right, p, q);
        if(left == null) return right;//左子树没查到，在右子树
        if(right == null) return left;//右子树没查到，在左子树
        return root;//分别在左右子树中
    }
}
```

情况 1. , 2. , 3. , 4. 的展开写法如下。

```java
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if(root == null || root == p || root == q) return root;
        TreeNode left = lowestCommonAncestor(root.left, p, q);
        TreeNode right = lowestCommonAncestor(root.right, p, q);
        if(left == null && right == null) return null; // 1.
        if(left == null) return right; // 3.
        if(right == null) return left; // 4.
        return root; // 2. if(left != null and right != null)
    }
}
```

时间复杂度 O(N) ： 其中 N 为二叉树节点数；最差情况下，需要递归遍历树的所有节点。

空间复杂度 O(N) ： 最差情况下，递归深度达到 N ，系统使用 O(N)大小的额外空间。

### [014] 和为s的连续正数序列

输入一个正整数 target ，输出所有和为 target 的连续正整数序列（至少含有两个数）。

序列内的数字由小到大排列，不同序列按照首个数字从小到大排列。

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

- 1 <= target <= 10^5

[方法一和方法二](https://leetcode-cn.com/problems/he-wei-sde-lian-xu-zheng-shu-xu-lie-lcof/solution/mian-shi-ti-57-ii-he-wei-sde-lian-xu-zheng-shu-x-2/)

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

对于这道题来说，数组就是正整数序列[1,2,3,…,n]。我们设滑动窗口的左边界为 i，右边界为 j，则滑动窗口框起来的是一个左闭右开区间 [i, j)。注意，为了编程的方便，滑动窗口一般表示成一个左闭右开区间。

滑动窗口的重要性质是：**窗口的左边界和右边界永远只能向右移动**，而不能向左移动。

在这道题中，我们关注的是滑动窗口中所有数的和。当滑动窗口的右边界向右移动时，也就是 j = j + 1，窗口中多了一个数字 j，窗口的和也就要加上 j。当滑动窗口的左边界向右移动时，也就是 i = i + 1，窗口中少了一个数字 i，窗口的和也就要减去 i。

![proof](https://pic.leetcode-cn.com/728c705889a672d5a85709cb3fd157216bb1a41dc377dcc125818d9e18b8dd55.jpg)

 ```java
class Solution {
    public int[][] findContinuousSequence(int target) {
        int i = 1; // 滑动窗口的左边界
        int j = 1; // 滑动窗口的右边界
        int sum = 0; // 滑动窗口中数字的和
        List<int[]> res = new ArrayList<>();

        while (i <= target / 2) {//i>target/2,再加上一个比它大的数，必然不符合条件
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
}
 ```

时间复杂度：由于两个指针移动均单调不减，且最多移动
$$
\lfloor\frac{\textit{target}}{2}\rfloor
$$
次，即方法一提到的枚举的上界，所以时间复杂度为 O(target) 。

空间复杂度：O(1) ，除了答案数组只需要常数的空间存放若干变量。

### [015] 二叉搜索树的最近公共祖先

> **[013] 二叉树的最近公共祖先**

给定一个二叉搜索树, 找到该树中两个指定节点的最近公共祖先。

百度百科中最近公共祖先的定义为：“对于有根树 T 的两个结点 p、q，最近公共祖先表示为一个结点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（一个节点也可以是它自己的祖先）。”

例如，给定如下二叉搜索树:  root = [6,2,8,0,4,7,9,null,null,3,5]

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/14/binarysearchtree_improved.png)

示例 1:

```
输入: root = [6,2,8,0,4,7,9,null,null,3,5], p = 2, q = 8
输出: 6 
解释: 节点 2 和节点 8 的最近公共祖先是 6。
```

示例 2:

```
输入: root = [6,2,8,0,4,7,9,null,null,3,5], p = 2, q = 4
输出: 2
解释: 节点 2 和节点 4 的最近公共祖先是 2, 因为根据定义最近公共祖先节点可以为节点本身。
```


说明:

- 所有节点的值都是唯一的。
- p、q 为不同节点且均存在于给定的二叉搜索树中。

方法一：递归

递推工作：

- 当 p, q 都在 root 的 右子树 中，则开启递归 root.right 并返回；
- 否则，当 p, q 都在 root 的 左子树 中，则开启递归 root.left 并返回；

返回值： 最近公共祖先 root 

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
        if(root.val < p.val && root.val < q.val)
            return lowestCommonAncestor(root.right, p, q);
        if(root.val > p.val && root.val > q.val)
            return lowestCommonAncestor(root.left, p, q);
        return root;
    }
}
```

- 时间复杂度 O(N) 

- 空间复杂度 O(N)

方法二：迭代

循环搜索： 当节点 root 为空时跳出；

- 当 p, q 都在 root的 右子树 中，则遍历至 root.right；
- 否则，当 p, q 都在 root 的 左子树 中，则遍历至 root.left；
- 否则，说明找到了 最近公共祖先 ，跳出t。

返回值： 最近公共祖先 root 。

```java
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        while(root != null) {
            if(root.val < p.val && root.val < q.val) // p,q 都在 root 的右子树中
                root = root.right; // 遍历至右子节点
            else if(root.val > p.val && root.val > q.val) // p,q 都在 root 的左子树中
                root = root.left; // 遍历至左子节点
            else break;
        }
        return root;
    }
}
```

优化：若可保证 p.val < q.val，则在循环中可减少判断条件。

```java
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if(p.val > q.val) { // 保证 p.val < q.val
            TreeNode tmp = p;
            p = q;
            q = tmp;
        }
        while(root != null) {
            if(root.val < p.val) // p,q 都在 root 的右子树中
                root = root.right; // 遍历至右子节点
            else if(root.val > q.val) // p,q 都在 root 的左子树中
                root = root.left; // 遍历至左子节点
            else break;
        }
        return root;
    }
}
```

- 时间复杂度 O(N) 

- 空间复杂度 O(1)

### [016] 从上到下打印二叉树 II  

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

- 节点总数 <= 1000

方法一：迭代

```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        LinkedList<TreeNode> queue = new LinkedList<>();
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

- 时间复杂度 O(N) 
- 空间复杂度 O(N)

方法二：递归

```java
public class Solution {
    private List<List<Integer>> ret;

    public List<List<Integer>> levelOrder(TreeNode root) {
        ret = new ArrayList<>();
        dfs(0, root);
        return ret;
    }

    private void dfs(int depth, TreeNode root) {
        if (root == null) {
            return;
        }
        if (ret.size() == depth) {
            ret.add(new ArrayList<>());
        }
        ret.get(depth).add(root.val);
        dfs(depth + 1, root.left);
        dfs(depth + 1, root.right);
    }
}
```

- 时间复杂度 O(N) 
- 空间复杂度 O(N)

### [017] 数组中出现次数超过一半的数字

数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。

你可以假设数组是非空的，并且给定的数组总是存在多数元素。

示例 1:

输入: [1, 2, 3, 2, 2, 2, 5, 4, 2]
输出: 2


限制：

1 <= 数组长度 <= 50000

方法一：哈希表

```java
class Solution {
    private Map<Integer, Integer> countNums(int[] nums) {
        Map<Integer, Integer> counts = new HashMap<Integer, Integer>();
        for (int num : nums) {
            if (!counts.containsKey(num)) {
                counts.put(num, 1);
            } else {
                counts.put(num, counts.get(num) + 1);
            }
        }
        return counts;
    }

    public int majorityElement(int[] nums) {
        Map<Integer, Integer> counts = countNums(nums);

        Map.Entry<Integer, Integer> majorityEntry = null;
        for (Map.Entry<Integer, Integer> entry : counts.entrySet()) {
            if (majorityEntry == null || entry.getValue() > majorityEntry.getValue()) {
                majorityEntry = entry;
            }
        }

        return majorityEntry.getKey();
    }
}
```

- 时间复杂度：O(n)

- 空间复杂度：O(n)

方法二：排序

```java
class Solution {
    public int majorityElement(int[] nums) {
        Arrays.sort(nums);
        return nums[nums.length / 2];
    }
}
```

- 时间复杂度：O(nlogn)

- 空间复杂度：O(logn)

方法三：随机化

由于一个给定的下标对应的数字很有可能是众数，我们随机挑选一个下标，检查它是否是众数，如果是就返回，否则继续随机挑选。

```java
class Solution {
    private int randRange(Random rand, int min, int max) {
        return rand.nextInt(max - min) + min;
    }

    private int countOccurences(int[] nums, int num) {
        int count = 0;
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] == num) {
                count++;
            }
        }
        return count;
    }

    public int majorityElement(int[] nums) {
        Random rand = new Random();

        int majorityCount = nums.length / 2;

        while (true) {
            int candidate = nums[randRange(rand, 0, nums.length)];
            if (countOccurences(nums, candidate) > majorityCount) {
                return candidate;
            }
        }
    }
}
```

- 时间复杂度：理论上最坏情况下的时间复杂度为O(∞)

- 空间复杂度：O(1)

方法四：分治

```java
class Solution {
    private int countInRange(int[] nums, int num, int lo, int hi) {
        int count = 0;
        for (int i = lo; i <= hi; i++) {
            if (nums[i] == num) {
                count++;
            }
        }
        return count;
    }

    private int majorityElementRec(int[] nums, int lo, int hi) {
        // base case; the only element in an array of size 1 is the majority
        // element.
        if (lo == hi) {
            return nums[lo];
        }

        // recurse on left and right halves of this slice.
        int mid = (hi - lo) / 2 + lo;
        int left = majorityElementRec(nums, lo, mid);
        int right = majorityElementRec(nums, mid + 1, hi);

        // if the two halves agree on the majority element, return it.
        if (left == right) {
            return left;
        }

        // otherwise, count each element and return the "winner".
        int leftCount = countInRange(nums, left, lo, hi);
        int rightCount = countInRange(nums, right, lo, hi);

        return leftCount > rightCount ? left : right;
    }

    public int majorityElement(int[] nums) {
        return majorityElementRec(nums, 0, nums.length - 1);
    }
}
```

- 时间复杂度：*O*(nlogn)

- 空间复杂度：O(logn)

方法五：Boyer-Moore 投票算法

```java
class Solution {
    public int majorityElement(int[] nums) {
        int count = 0;
        Integer candidate = null;

        for (int num : nums) {
            if (count == 0) {
                candidate = num;
            }
            count += (num == candidate) ? 1 : -1;
        }

        return candidate;
    }
}
```

- 时间复杂度：O(n)
- 空间复杂度：O(1)

### [018] 数组中重复的数字  

找出数组中重复的数字。


在一个长度为 n 的数组 nums 里的所有数字都在 0～n-1 的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。

示例 1：

```
输入：
[2, 3, 1, 0, 2, 5, 3]
输出：2 或 3 
```


限制：

- 2 <= n <= 100000

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

如果没有重复数字，那么正常排序后，数字i应该在下标为i的位置

思路是重头扫描数组，遇到下标为i的数字如果不是i的话，（假设为m),那么我们就拿与下标m的数字交换。在交换过程中，如果有重复的数字发生，那么终止返回ture 

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

### [019] 和为s的两个数字  

输入一个递增排序的数组和一个数字s，在数组中查找两个数，使得它们的和正好是s。如果有多对数字的和等于s，则输出任意一对即可。

 示例 1：

```
输入：nums = [2,7,11,15], target = 9
输出：[2,7] 或者 [7,2]
```

示例 2：

```
输入：nums = [10,26,30,31,47,60], target = 40
输出：[10,30] 或者 [30,10]
```

限制：

- 1 <= nums.length <= 10^5
- 1 <= nums[i] <= 10^6

方法一：

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        int i = 0, j = nums.length - 1;
        while(i < j) {
            int s = nums[i] + nums[j];
            if(s < target) i++;
            else if(s > target) j--;
            else return new int[] { nums[i], nums[j] };
        }
        return new int[0];
    }
}
```

- 时间复杂度 O(N)
- 空间复杂度 O(1)

### [020] 调整数组顺序使奇数位于偶数前面

输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有奇数位于数组的前半部分，所有偶数位于数组的后半部分。

示例：

```
输入：nums = [1,2,3,4]
输出：[1,3,2,4] 
注：[3,1,2,4] 也是正确的答案之一。
```


提示：

- 1 <= nums.length <= 50000
- 1 <= nums[i] <= 10000

方法一：

```java
class Solution {
    public int[] exchange(int[] nums) {
        int i = 0, j = nums.length - 1, tmp;
        while(i < j) {
            while(i < j && (nums[i] & 1) == 1) i++;
            while(i < j && (nums[j] & 1) == 0) j--;
            tmp = nums[i];
            nums[i] = nums[j];
            nums[j] = tmp;
        }
        return nums;
    }
}
```

- 时间复杂度 O(N) 
- 空间复杂度 O(1)

### [021] 圆圈中最后剩下的数字

0,1...n-1这n个数字排成一个圆圈，从数字0开始，每次从这个圆圈里删除第m个数字。求出这个圆圈里剩下的最后一个数字。

例如，0、1、2、3、4这5个数字组成一个圆圈，从数字0开始每次删除第3个数字，则删除的前4个数字依次是2、0、4、1，因此最后剩下的数字是3。

 示例 1：

```
输入: n = 5, m = 3
输出: 3
```

示例 2：

```
输入: n = 10, m = 17
输出: 2
```


限制：

- 1 <= n <= 10^5
- 1 <= m <= 10^6

方法一：暴力

```java
class Solution {
    public int lastRemaining(int n, int m) {
        ArrayList<Integer> list = new ArrayList<>(n);
        for (int i = 0; i < n; i++) {
            list.add(i);
        }
        int idx = 0;
        while (n > 1) {
            idx = (idx + m - 1) % n;
            list.remove(idx);
            n--;
        }
        return list.get(0);
    }
}
```

- 时间复杂度 O(N^2) 
- 空间复杂度 O(N)

方法二：数学

![image.png](https://pic.leetcode-cn.com/9dda886441be8d249abb76e35f53f29fd6e780718d4aca2ee3c78f947fb76e75-image.png)

删除的位置 = (当前index + m) % 上一轮剩余数字的个数。

**递归**

```java
class Solution {
    public int lastRemaining(int n, int m) {
        return f(n, m);
    }

    public int f(int n, int m) {
        if (n == 1) {
            return 0;
        }
        int x = f(n - 1, m);
        return (m + x) % n;
    }
}
```

时间复杂度：O(n) 

空间复杂度：O(n) 

**迭代**

```java
class Solution {
    public int lastRemaining(int n, int m) {
        int f = 0;
        for (int i = 2;  i <= n; i++) {
            f = (m + f) % i;
        }
        return f;
    }
}
```

复杂度分析

- 时间复杂度：O(n) 

- 空间复杂度：O(1) 

### [022] 两个链表的第一个公共节点

输入两个链表，找出它们的第一个公共节点。

如下面的两个链表：

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/14/160_statement.png)

在节点 c1 开始相交。

 

示例 1：

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/14/160_example_1.png)

```
输入：intersectVal = 8, listA = [4,1,8,4,5], listB = [5,0,1,8,4,5], skipA = 2, skipB = 3
输出：Reference of the node with value = 8
输入解释：相交节点的值为 8 （注意，如果两个链表相交则不能为 0）。从各自的表头开始算起，链表 A 为 [4,1,8,4,5]，链表 B 为 [5,0,1,8,4,5]。在 A 中，相交节点前有 2 个节点；在 B 中，相交节点前有 3 个节点。
```


示例 2：

[![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/14/160_example_2.png)](https://assets.leetcode.com/uploads/2018/12/13/160_example_2.png)

```
输入：intersectVal = 2, listA = [0,9,1,2,4], listB = [3,2,4], skipA = 3, skipB = 1
输出：Reference of the node with value = 2
输入解释：相交节点的值为 2 （注意，如果两个链表相交则不能为 0）。从各自的表头开始算起，链表 A 为 [0,9,1,2,4]，链表 B 为 [3,2,4]。在 A 中，相交节点前有 3 个节点；在 B 中，相交节点前有 1 个节点。
```


示例 3：

[![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/14/160_example_3.png)](https://assets.leetcode.com/uploads/2018/12/13/160_example_3.png)

```
输入：intersectVal = 0, listA = [2,6,4], listB = [1,5], skipA = 3, skipB = 2
输出：null
输入解释：从各自的表头开始算起，链表 A 为 [2,6,4]，链表 B 为 [1,5]。由于这两个链表不相交，所以 intersectVal 必须为 0，而 skipA 和 skipB 可以是任意值。
解释：这两个链表不相交，因此返回 null。
```


注意：

- 如果两个链表没有交点，返回 null.
- 在返回结果后，两个链表仍须保持原有的结构。
- 可假定整个链表结构中没有循环。
- 程序尽量满足 O(n) 时间复杂度，且仅用 O(1) 内存。

方法一: 暴力法

```java
public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
    if(headA==null||headB==null){
        return null;
    }
    ListNode flagA=headA;
    do {
        ListNode flagB=headB;
        do {
            if(flagA==flagB) {
                return flagA;
            }else {
                flagB=flagB.next;
            }

        }while(flagB!=null);
        flagA=flagA.next;
    }while(flagA!=null);
    return null;
}//暴力法
```

- 时间复杂度 : (m*n) 
- 空间复杂度 : O(1) 

方法二: 哈希表法

```java
public ListNode getIntersectionNode1(ListNode headA, ListNode headB) {
    if(headA==null||headB==null) {
        return null;
    }
    HashMap<ListNode, Integer> nodeOfHeadA=new HashMap<ListNode, Integer>();
    ListNode pA=headA;
    while(pA!=null) {
        if(!nodeOfHeadA.containsKey(pA)) {
            nodeOfHeadA.put(pA,pA.val);
        }
        pA=pA.next;
    }
    ListNode pB=headB;
    while(pB!=null) {
        if(nodeOfHeadA.containsKey(pB)) {
            return pB;
        }
        pB=pB.next;
    }
    return null;
}//哈希表法

```

- 时间复杂度 : O(m+n) 
- 空间复杂度 : O(m) 或 O(n)。

**方法三**：双指针法，消除长度差，拼接两链表

创建两个指针 pA 和 pB，分别初始化为链表 A 和 B 的头结点。然后让它们向后逐结点遍历。

当 pA 到达链表的尾部时，将它重定位到链表 B 的头结点; 类似的，当 pB 到达链表的尾部时，将它重定位到链表 A 的头结点。

若在某一时刻 pA 和 pB 相遇，则 pA/pB 为相交结点。

```
pA:1->2->3->4->5->6->null->9->5->6->null
pB:9->5->6->null->1->2->3->4->5->6->null
```

 ```java
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        if (headA == null || headB == null) return null;
        ListNode pA = headA, pB = headB;
        while (pA != pB) {
            pA = pA == null ? headB : pA.next;
            pB = pB == null ? headA : pB.next;
        }
        return pA;
    }
}
 ```

- 时间复杂度 : O(m+n) 
- 空间复杂度 : O(1)

### [023] 第一个只出现一次的字符

在字符串 s 中找出第一个只出现一次的字符。如果没有，返回一个单空格。 s 只包含小写字母。

示例:

```
s = "abaccdeff"
返回 "b"

s = "" 
返回 " "
```


限制：

0 <= s 的长度 <= 50000

方法一：哈希表

遍历字符串 s ，使用哈希表统计 “各字符数量是否 > 1>1 ”。

再遍历字符串 s ，在哈希表中找到首个 “数量为 1的字符”，并返回。

```java
class Solution {
    public char firstUniqChar(String s) {
        HashMap<Character, Boolean> dic = new HashMap<>();
        char[] sc = s.toCharArray();
        for(char c : sc)
            dic.put(c, !dic.containsKey(c));
        for(char c : sc)
            if(dic.get(c)) return c;
        return ' ';
    }
}
```

- 时间复杂度 O(N)

- 空间复杂度 O(1)

方法二：有序哈希表

在哈希表的基础上，有序哈希表中的键值对是 **按照插入顺序排序** 的。基于此，可通过遍历有序哈希表，实现搜索首个 “数量为 11 的字符”。

相比于方法一，方法二减少了第二轮遍历的循环次数。当字符串很长（重复字符很多）时，方法二则效率更高。

```java
class Solution {
    public char firstUniqChar(String s) {
        Map<Character, Boolean> dic = new LinkedHashMap<>();
        char[] sc = s.toCharArray();
        for(char c : sc)
            dic.put(c, !dic.containsKey(c));
        for(Map.Entry<Character, Boolean> d : dic.entrySet()){
           if(d.getValue()) return d.getKey();
        }
        return ' ';
    }
}
```

- 时间复杂度 O(N)

- 空间复杂度 O(1)

### [024] 连续子数组的最大和

输入一个整型数组，数组中的一个或连续多个整数组成一个子数组。求所有子数组的和的最大值。

要求时间复杂度为O(n)。

 示例1:

```
输入: nums = [-2,1,-3,4,-1,2,1,-5,4]
输出: 6
解释: 连续子数组 [4,-1,2,1] 的和最大，为 6。
```


提示：

- 1 <= arr.length <= 10^5
- -100 <= arr[i] <= 100

给定一个整数数组 nums ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

进阶:

如果你已经实现复杂度为 O(n) 的解法，尝试使用更为精妙的分治法求解。

[题解](https://leetcode-cn.com/problems/maximum-subarray/solution/)

| 常见解法 | 时间复杂度 | 空间复杂度 |
| -------- | ---------- | ---------- |
| 暴力搜索 | O(N^2)     | O(1)       |
| 分治思想 | O(NlogN)   | O(logN)    |
| 动态规划 | O(N)       | O(1)       |

方法一：动态规划

![Picture1.png](https://pic.leetcode-cn.com/8fec91e89a69d8695be2974de14b74905fcd60393921492bbe0338b0a628fd9a-Picture1.png)



```java
class Solution {
    public int maxSubArray(int[] nums) {
        int pre = 0, maxAns = nums[0];
        for (int x : nums) {
            pre = Math.max(pre + x, x);
            maxAns = Math.max(maxAns, pre);
        }
        return maxAns;
    }
}
```

- 时间复杂度：O(n) 
- 空间复杂度：O(1) 

方法二：分治 ★

```java
class Solution {
    public class Status {
        public int lSum, rSum, mSum, iSum;

        public Status(int lSum, int rSum, int mSum, int iSum) {
            this.lSum = lSum;
            this.rSum = rSum;
            this.mSum = mSum;
            this.iSum = iSum;
        }
    }

    public int maxSubArray(int[] nums) {
        return getInfo(nums, 0, nums.length - 1).mSum;
    }

    public Status getInfo(int[] a, int l, int r) {
        if (l == r) {
            return new Status(a[l], a[l], a[l], a[l]);
        }
        int m = (l + r) >> 1;
        Status lSub = getInfo(a, l, m);
        Status rSub = getInfo(a, m + 1, r);
        return pushUp(lSub, rSub);
    }

    public Status pushUp(Status l, Status r) {
        int iSum = l.iSum + r.iSum;
        int lSum = Math.max(l.lSum, l.iSum + r.lSum);
        int rSum = Math.max(r.rSum, r.iSum + l.rSum);
        int mSum = Math.max(Math.max(l.mSum, r.mSum), l.rSum + r.lSum);
        return new Status(lSum, rSum, mSum, iSum);
    }
}
```

### [025] 平衡二叉树

输入一棵二叉树的根节点，判断该树是不是平衡二叉树。如果某二叉树中任意节点的左右子树的深度相差不超过1，那么它就是一棵平衡二叉树。

 示例 1:

给定二叉树 [3,9,20,null,null,15,7]

    	3
       / \
      9  20
        /  \
       15   7

返回 true 。

示例 2:

给定二叉树 [1,2,2,3,3,null,null,4,4]

           1
          / \
         2   2
        / \
      3   3
      / \
     4   4
返回 `false` 。

**此树的深度** 等于 **左子树的深度** 与 **右子树的深度** 中的 **最大值** +1

方法一：后序遍历 + 剪枝 （从底至顶）

思路是对二叉树做后序遍历，从底至顶返回子树深度，若判定某子树不是平衡树则 “剪枝” ，直接向上返回。

`recur(root) `函数：

- 返回值：

1. 当节点root 左 / 右子树的深度差 ≤1 ：则返回当前子树的深度，即节点 root 的左 / 右子树的深度最大值 +1 （ `max(left, right) + 1` ）；
2. 当节点root 左 / 右子树的深度差 > 2>2 ：则返回 -1−1 ，代表 此子树不是平衡树 。

- 终止条件：

1. 当 root 为空：说明越过叶节点，因此返回高度 00 ；
2. 当左（右）子树深度为 -1−1 ：代表此树的 左（右）子树 不是平衡树，因此剪枝，直接返回 -1−1 ；

`isBalanced(root) `函数：

返回值： 若 recur(root) != -1 ，则说明此树平衡，返回 true ； 否则返回  false 。

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
    public boolean isBalanced(TreeNode root) {
        return recur(root) != -1;
    }

    private int recur(TreeNode root) {
        if (root == null) return 0;
        int left = recur(root.left);
        if(left == -1) return -1;
        int right = recur(root.right);
        if(right == -1) return -1;
        return Math.abs(left - right) < 2 ? Math.max(left, right) + 1 : -1;
    }
}
```

- 时间复杂度 O(N) 

- 空间复杂度 O(N) 

方法二：先序遍历 + 判断深度 （从顶至底）

通过比较某子树的左右子树的深度差 abs(depth(root.left) - depth(root.right)) <= 1 是否成立，来判断某子树是否是二叉平衡树。若所有子树都平衡，则此树平衡。

**算法流程：**

**isBalanced(root) 函数**： 判断树 root 是否平衡

- 特例处理： 若树根节点 root 为空，则直接返回 truetrue ；
- 返回值： 所有子树都需要满足平衡树性质，因此以下三者使用与逻辑 \&\&&& 连接；
  1. abs(self.depth(root.left) - self.depth(root.right)) <= 1 ：判断 当前子树 是否是平衡树；
  2. self.isBalanced(root.left) ： 先序遍历递归，判断 当前子树的左子树 是否是平衡树；
  3. self.isBalanced(root.right) ： 先序遍历递归，判断 当前子树的右子树 是否是平衡树；

**depth(root) 函数：** 计算树 root 的深度

- 终止条件： 当 root 为空，即越过叶子节点，则返回高度 0 ；
- 返回值： 返回左 / 右子树的深度的最大值 +1 。

```java
class Solution {
    public boolean isBalanced(TreeNode root) {
        if (root == null) return true;
        return Math.abs(depth(root.left) - depth(root.right)) <= 1 && isBalanced(root.left) && isBalanced(root.right);
    }

    private int depth(TreeNode root) {
        if (root == null) return 0;
        return Math.max(depth(root.left), depth(root.right)) + 1;
    }
}
```

- 时间复杂度 O(NlogN) 

- 空间复杂度 O(N) 

### [026] 删除链表的节点

给定单向链表的头指针和一个要删除的节点的值，定义一个函数删除该节点。

返回删除后的链表的头节点。

注意：此题对比原题有改动

示例 1:

```
输入: head = [4,5,1,9], val = 5
输出: [4,1,9]
解释: 给定你链表中值为 5 的第二个节点，那么在调用了你的函数之后，该链表应变为 4 -> 1 -> 9.
```

示例 2:

```
输入: head = [4,5,1,9], val = 1
输出: [4,5,9]
解释: 给定你链表中值为 1 的第三个节点，那么在调用了你的函数之后，该链表应变为 4 -> 5 -> 9.
```


说明：

- 题目保证链表中节点的值互不相同
- 若使用 C 或 C++ 语言，你不需要 free 或 delete 被删除的节点

方法一：递归

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
    public ListNode deleteNode(ListNode head, int val) {
        if (head == null)
            return head;
        if (head.val == val)
            return head.next;
        head.next = deleteNode(head.next, val);
        return head;
    }
}
```

- 时间复杂度 O(N) 

- 空间复杂度 O(N) 

方法二：

```java
class Solution {
    public ListNode deleteNode(ListNode head, int val) {
        if (head == null) return null;
        if (head.val == val) return head.next;
        ListNode cur = head;
        ///找到要删除结点的上一个结点
        while (cur.next != null && cur.next.val != val)
            cur = cur.next;
        if (cur.next != null)
            cur.next = cur.next.next;
        return head;
    }
}
```

注意删除节点是最后一个节点

- 时间复杂度 O(N) 

- 空间复杂度 O(1) 

### [027] 对称的二叉树

请实现一个函数，用来判断一棵二叉树是不是对称的。如果一棵二叉树和它的镜像一样，那么它是对称的。

例如，二叉树 [1,2,2,3,4,4,3] 是对称的。

    	1
       / \
      2   2
     / \ / \
    3  4 4  3

但是下面这个 [1,2,2,null,3,null,3] 则不是镜像对称的:

    	1
       / \
      2   2
       \   \
       3    3
示例 1：

```
输入：root = [1,2,2,3,4,4,3]
输出：true
```

示例 2：

```
输入：root = [1,2,2,null,3,null,3]
输出：false
```


限制：

- 0 <= 节点个数 <= 1000

方法一：递归

递归函数，通过「同步移动」两个指针的方法来遍历这棵树，p 指针和 q 指针一开始都指向这棵树的根，随后 p 右移时，q 左移，p 左移时，q 右移。每次检查当前 p 和 q 节点的值是否相等，如果相等再判断左右子树是否对称。

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
    public boolean isSymmetric(TreeNode root) {
        return check(root, root);
    }

    public boolean check(TreeNode p, TreeNode q) {
        if (p == null && q == null) {
            return true;
        }
        if (p == null || q == null) {
            return false;
        }
        return p.val == q.val && check(p.left, q.right) && check(p.right, q.left);
    }
}
```

- 时间复杂度：这里遍历了这棵树，渐进时间复杂度为 O(n)
- 空间复杂度：这里的空间复杂度和递归使用的栈空间有关，这里递归层数不超过 n，故渐进空间复杂度为 O(n)

方法二：迭代

首先我们引入一个队列，这是把递归程序改写成迭代程序的常用方法。初始化时我们把根节点入队两次。每次提取两个结点并比较它们的值（队列中每两个连续的结点应该是相等的，而且它们的子树互为镜像），然后将两个结点的左右子结点按相反的顺序插入队列中。当队列为空时，或者我们检测到树不对称（即从队列中取出两个不相等的连续结点）时，该算法结束。

```java
class Solution {
    public boolean isSymmetric(TreeNode root) {
        return check(root, root);
    }

    public boolean check(TreeNode u, TreeNode v) {
        Queue<TreeNode> q = new LinkedList<TreeNode>();
        q.offer(u);
        q.offer(v);
        while (!q.isEmpty()) {
            u = q.poll();
            v = q.poll();
            if (u == null && v == null) {
                continue;
            }
            if ((u == null || v == null) || (u.val != v.val)) {
                return false;
            }

            q.offer(u.left);
            q.offer(v.right);

            q.offer(u.right);
            q.offer(v.left);
        }
        return true;
    }
}
```

- 时间复杂度：O(n) 
- 空间复杂度：这里需要用一个队列来维护节点，每个节点最多进队一次，出队一次，队列中最多不会超过 n 个点，故渐进空间复杂度为 O(n) 

### [028] 包含min函数的栈

定义栈的数据结构，请在该类型中实现一个能够得到栈的最小元素的 min 函数在该栈中，调用 min、push 及 pop 的时间复杂度都是 O(1)。

 示例:

```
MinStack minStack = new MinStack();
minStack.push(-2);
minStack.push(0);
minStack.push(-3);
minStack.min();   --> 返回 -3.
minStack.pop();
minStack.top();      --> 返回 0.
minStack.min();   --> 返回 -2.
```


提示：

- 各函数的调用总次数不超过 20000 次

方法一：辅助栈

普通栈的 push() 和 pop() 函数的复杂度为 O(1) ；而获取栈最小值 min() 函数需要遍历整个栈，复杂度为 O(N)。

本题难点： 将 min() 函数复杂度降为 O(1)，可通过建立辅助栈实现；

- 数据栈 A ： 栈 A 用于存储所有元素，保证入栈 push() 函数、出栈 pop() 函数、获取栈顶 top() 函数的正常逻辑。
- 辅助栈 B ： 栈 B 中存储栈 A 中所有 非严格降序 的元素，则栈 A 中的最小元素始终对应栈 B 的栈顶元素，即 min() 函数只需返回栈 BB 的栈顶元素即可。

![Picture1.png](https://pic.leetcode-cn.com/f31f4b7f5e91d46ea610b6685c593e12bf798a9b8336b0560b6b520956dd5272-Picture1.png)



```java
class MinStack {
    Stack<Integer> A, B;
    public MinStack() {
        A = new Stack<>();
        B = new Stack<>();
    }
    public void push(int x) {
        A.add(x);
        if(B.empty() || B.peek() >= x)
            B.add(x);
    }
    public void pop() {
        if(A.pop().equals(B.peek()))
            B.pop();
    }
    public int top() {
        return A.peek();
    }
    public int min() {
        return B.peek();
    }
}
```

- 时间复杂度 O(1) ： push(), pop(), top(), min() 四个函数的时间复杂度均为常数级别。
- 空间复杂度 O(N) ： 当共有 N 个待入栈元素时，辅助栈 B 最差情况下存储 N 个元素，使用 O(N) 额外空间。

方法二：辅助类

```java
class MinStack {
    //链表头，相当于栈顶
    private ListNode head;

    //压栈，需要判断栈是否为空
    public void push(int x) {
        if (empty())
            head = new ListNode(x, x, null);
        else
            head = new ListNode(x, Math.min(x, head.min), head);
    }

    //出栈，相当于把链表头删除
    public void pop() {
        if (empty())
            throw new IllegalStateException("栈为空……");
        head = head.next;
    }

    //栈顶的值也就是链表头的值
    public int top() {
        if (empty())
            throw new IllegalStateException("栈为空……");
        return head.val;
    }

    //链表中头结点保存的是整个链表最小的值，所以返回head.min也就是
    //相当于返回栈中最小的值
    public int min() {
        if (empty())
            throw new IllegalStateException("栈为空……");
        return head.min;
    }

    //判断栈是否为空
    private boolean empty() {
        return head == null;
    }
}

class ListNode {
    public int val;
    public int min;//最小值
    public ListNode next;

    public ListNode(int val, int min, ListNode next) {
        this.val = val;
        this.min = min;
        this.next = next;
    }
}
```

- 时间复杂度 O(1) 
- 空间复杂度 O(N) 

### [029] 最小的k个数

输入整数数组 arr ，找出其中最小的 k 个数。例如，输入4、5、1、6、2、7、3、8这8个数字，则最小的4个数字是1、2、3、4。

 示例 1：

```
输入：arr = [3,2,1], k = 2
输出：[1,2] 或者 [2,1]
```

示例 2：

```
输入：arr = [0,1,2,1], k = 1
输出：[0]
```


限制：

- 0 <= k <= arr.length <= 10000
- 0 <= arr[i] <= 10000

方法一：排序

```java
class Solution {
    public int[] getLeastNumbers(int[] arr, int k) {
        int[] vec = new int[k];
        Arrays.sort(arr);
        for (int i = 0; i < k; ++i) {
            vec[i] = arr[i];
        }
        return vec;
    }
}
```

时间复杂度：O(nlogn)，其中 n 是数组 arr 的长度。算法的时间复杂度即排序的时间复杂度。

空间复杂度：O(logn)，排序所需额外的空间复杂度为 O(logn)

方法二：堆

```java
class Solution {
    public int[] getLeastNumbers(int[] arr, int k) {
        int[] vec = new int[k];
        if (k == 0) { // 排除 0 的情况
            return vec;
        }
        PriorityQueue<Integer> queue = new PriorityQueue<Integer>(new Comparator<Integer>() {
            public int compare(Integer num1, Integer num2) {
                return num2 - num1;
            }
        });
        for (int i = 0; i < k; ++i) {
            queue.offer(arr[i]);
        }
        for (int i = k; i < arr.length; ++i) {
            if (queue.peek() > arr[i]) {
                queue.poll();
                queue.offer(arr[i]);
            }
        }
        for (int i = 0; i < k; ++i) {
            vec[i] = queue.poll();
        }
        return vec;
    }
}
```

时间复杂度：O(nlogk)，其中 n 是数组 arr 的长度。由于大根堆实时维护前 k 小值，所以插入删除都是 O(logk) 的时间复杂度，最坏情况下数组里 n 个数都会插入，所以一共需要 O(nlogk) 的时间复杂度。

空间复杂度：O(k)，因为大根堆里最多 k 个数。

方法三：快排思想

```java
class Solution {
    public int[] getLeastNumbers(int[] arr, int k) {
        randomizedSelected(arr, 0, arr.length - 1, k);
        int[] vec = new int[k];
        for (int i = 0; i < k; ++i) {
            vec[i] = arr[i];
        }
        return vec;
    }

    public void randomizedSelected(int[] arr, int l, int r, int k) {
        if (l >= r) {
            return;
        }
        int pos = randomizedPartition(arr, l, r);
        int num = pos - l + 1;
        if (k == num) {
            return;
        } else if (k < num) {
            randomizedSelected(arr, l, pos - 1, k);
        } else {
            randomizedSelected(arr, pos + 1, r, k - num);
        }
    }

    // 基于随机的划分
    public int randomizedPartition(int[] nums, int l, int r) {
        int i = new Random().nextInt(r - l + 1) + l;
        swap(nums, r, i);
        return partition(nums, l, r);
    }

    public int partition(int[] nums, int l, int r) {
        int pivot = nums[r];
        int i = l - 1;
        for (int j = l; j <= r - 1; ++j) {
            if (nums[j] <= pivot) {
                i = i + 1;
                swap(nums, i, j);
            }
        }
        swap(nums, i + 1, r);
        return i + 1;
    }

    private void swap(int[] nums, int i, int j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
}
```

- 时间复杂度：期望为 O*(*n) 。最坏情况下的时间复杂度为 O(n^2) 

- 空间复杂度：期望为 O(logn)。最坏情况下的空间复杂度为 O(n) 

方法四：

```java
class Solution {
    public int[] getLeastNumbers(int[] arr, int k) {
        if (k == 0 || arr.length == 0) {
            return new int[0];
        }
        // 统计每个数字出现的次数
        int[] counter = new int[10001];
        for (int num: arr) {
            counter[num]++;
        }
        // 根据counter数组从头找出k个数作为返回结果
        int[] res = new int[k];
        int idx = 0;
        for (int num = 0; num < counter.length; num++) {
            while (counter[num]-- > 0 && idx < k) {
                res[idx++] = num;
            }
            if (idx == k) {
                break;
            }
        }
        return res;
    }
}
```

- 时间复杂度：O(N)

- 空间复杂度：O(N)

### [030] 不用加减乘除做加法

写一个函数，求两个整数之和，要求在函数体内不得使用 “+”、“-”、“*”、“/” 四则运算符号。

示例:

```
输入: a = 1, b = 1
输出: 2
```


提示：

- a, b 均可能是负数或 0
- 结果不会溢出 32 位整数

方法一：位运算

设两数字的二进制形式 a, b，其求和 s = a + b，a(i) 代表 a 的二进制第 i 位，则分为以下四种情况：

| *a*(*i*) | b(i)*b*(*i*) | 无进位和 n(i) | 进位 c(i+1) |
| -------- | ------------ | ------------- | ----------- |
| 0        | 0            | 0             | 0           |
| 0        | 1            | 1             | 0           |
| 1        | 0            | 1             | 0           |
| 1        | 1            | 0             | 1           |

观察发现，**无进位和** 与 **异或运算** 规律相同，**进位** 和 **与运算** 规律相同（并需左移一位）。因此，无进位和 n 与进位 c 的计算公式如下；
$$
{ 
n=a⊕b
}
$$

$$
c=a\&b<<1
$$

（和 s ）=（非进位和 n ）+（进位 c ）。即可将 s = a + b转化为：

$$
s = a + b \Rightarrow s = n + c
$$
循环求 n 和 c ，直至进位 c = 0  ；此时 s = n ，返回 n  即可。

![Picture1.png](https://pic.leetcode-cn.com/56d56524d8d2b1318f78e209fffe0e266f97631178f6bfd627db85fcd2503205-Picture1.png)

Q ： 若数字 a 和 bb中有负数，则变成了减法，如何处理？
A ： 在计算机系统中，数值一律用 补码 来表示和存储。补码的优势： 加法、减法可以统一处理（CPU只有加法器）。因此，以上方法 同时适用于正数和负数的加法 。

```java
class Solution {
    public int add(int a, int b) {
        while(b != 0) { // 当进位为 0 时跳出
            int c = (a & b) << 1;  // c = 进位
            a ^= b; // a = 非进位和
            b = c; // b = 进位
        }
        return a;
    }
}
```

递归写法

```java
class Solution {
    public int add(int a, int b) {
        if (b == 0) {
            return a;
        }
        // 转换成非进位和 + 进位
        return add(a ^ b, (a & b) << 1);
    }
}
```

时间复杂度 O(1) ： 最差情况下（例如 a =a= 0x7fffffff , b = 1b=1 时），需循环 32 次，使用 O(1) 时间；每轮中的常数次位操作使用 O(1)时间。
空间复杂度 O(1) ： 使用常数大小的额外空间。

### [031] 在排序数组中查找数字 I

统计一个数字在排序数组中出现的次数。

示例 1:

```
输入: nums = [5,7,7,8,8,10], target = 8
输出: 2
```

示例 2:

```
输入: nums = [5,7,7,8,8,10], target = 6
输出: 0
```


限制：

- 0 <= 数组长度 <= 50000

方法一：二分法迭代

![Picture1.png](https://pic.leetcode-cn.com/b4521d9ba346cad9e382017d1abd1db2304b4521d4f2d839c32d0ecff17a9c0d-Picture1.png)

- 初始化： 左边界 `i = 0` ，右边界` j = len(nums) - 1` 。
- 循环二分： 当闭区间 `[i, j] `无元素时跳出；
  - 计算中点 `m = (i + j) / 2`（向下取整）；
  - 若 `nums[m] < target` ，则 target在闭区间` [m + 1, j] `中，因此执行` i = m + 1`；
  - 若 `nums[m] > target` ，则 target 在闭区间` [i, m - 1]`中，因此执行 `j = m - 1`；
  - 若 `nums[m] = target` ，则右边界 right 在闭区间` [m+1, j] `中；左边界 left 在闭区间` [i, m-1] `中。因此分为以下两种情况：
    - 若查找 右边界 right ，则执行` i = m + 1` ；（跳出时 i 指向右边界）
    - 若查找 左边界 left ，则执行` j = m - 1 `；（跳出时 j 指向左边界）
- 返回值： 应用两次二分，分别查找 right 和 left ，最终返回` right - left - 1`即可。

效率优化：

> 以下优化基于：查找完右边界 right = i 后，则 nums[j]指向最右边的 target （若存在）。

- 查找完右边界后，可用 `nums[j] = j`判断数组中是否包含 target ，若不包含则直接提前返回 0 ，无需后续查找左边界。
- 查找完右边界后，左边界 left 一定在闭区间` [0, j][0,j] `中，因此直接从此区间开始二分查找即可。

```java
class Solution {
    public int search(int[] nums, int target) {
        // 搜索右边界 right
        int i = 0, j = nums.length - 1;
        while(i <= j) {
            int m = (i + j) / 2;
            if(nums[m] <= target) i = m + 1;
            else j = m - 1;
        }
        int right = i;
        // 若数组中无 target ，则提前返回
        if(j >= 0 && nums[j] != target) return 0;
        // 搜索左边界 right
        i = 0; j = nums.length - 1;
        while(i <= j) {
            int m = (i + j) / 2;
            if(nums[m] < target) i = m + 1;
            else j = m - 1;
        }
        int left = j;
        return right - left - 1;
    }
}
```

- 时间复杂度：o(logN)

- 空间复杂度：o(1))

方法二：二分法递归

![Picture2.png](https://pic.leetcode-cn.com/bf124fb9feff173309e2a0c3d36d2a76d1a0a46cb34a78a5776ac255fd1fde1d-Picture2.png)

```java
class Solution {
    public int search(int[] nums, int target) {
        return helper(nums, target) - helper(nums, target - 1);
    }
    int helper(int[] nums, int tar) {
        int i = 0, j = nums.length - 1;
        while(i <= j) {
            int m = (i + j) / 2;
            if(nums[m] <= tar) i = m + 1;
            else j = m - 1;
        }
        return i;
    }
}
```

### [032] 旋转数组的最小数字

把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。输入一个递增排序的数组的一个旋转，输出旋转数组的最小元素。例如，数组 [3,4,5,1,2] 为 [1,2,3,4,5] 的一个旋转，该数组的最小值为1。  

示例 1：

```
输入：[3,4,5,1,2]
输出：1
```

示例 2：

```
输入：[2,2,2,0,1]
输出：0
```



方法一：二分

![Picture1.png](https://pic.leetcode-cn.com/1599404042-JMvjtL-Picture1.png)

算法流程：

- 初始化： 声明 i, j 双指针分别指向 nums 数组左右两端；
- 循环二分： 设` m = (i + j) / 2 `为每次二分的中点（ "/" 代表向下取整除法，因此恒有` i≤m<j `），可分为以下三种情况：
  - 当`nums[m] > nums[j]` 时： m 一定在 左排序数组 中，即旋转点 x 一定在` [m + 1, j] `闭区间内，因此执行` i = m + 1`；
  - 当 `nums[m] < nums[j]`时： m 一定在 右排序数组 中，即旋转点 x 一定在`[i, m] `闭区间内，因此执行` j = m`；
  - 当` nums[m] = nums[j] `时： 无法判断 m 在哪个排序数组中，即无法判断旋转点 x 在` [i, m] `还是` [m + 1, j] `区间中。解决方案： 执行`j = j - 1 `缩小判断范围。
- 返回值： 当 i = j 时跳出二分循环，并返回 旋转点的值 nums[i] 即可。

```java
class Solution {
    public int minArray(int[] numbers) {
        int i = 0, j = numbers.length - 1;
        while (i < j) {
            int m = (i + j) / 2;
            if (numbers[m] > numbers[j]) i = m + 1;
            else if (numbers[m] < numbers[j]) j = m;
            else j--;
        }
        return numbers[i];
    }
}
```

- 时间复杂度 O(log2 N)： 在特例情况下，会退化到 O(N)
- 空间复杂度 O(1) 

当出现` nums[m] = nums[j] `时，一定有区间` [i, m] `内所有元素相等 或 区间` [m, j] `内所有元素相等（或两者皆满足）。对于寻找此类数组的最小值问题，可直接放弃二分查找，而使用线性查找替代。

```java
class Solution {
    public int minArray(int[] numbers) {
        int i = 0, j = numbers.length - 1;
        while (i < j) {
            int m = (i + j) / 2;
            if (numbers[m] > numbers[j]) i = m + 1;
            else if (numbers[m] < numbers[j]) j = m;
            else {
                int x = i;
                for(int k = i + 1; k < j; k++) {
                    if(numbers[k] < numbers[x]) x = k;
                }
                return numbers[x];
            }
        }
        return numbers[i];
    }
}
```

### [033] 扑克牌中的顺子

从扑克牌中随机抽5张牌，判断是不是一个顺子，即这5张牌是不是连续的。2～10为数字本身，A为1，J为11，Q为12，K为13，而大、小王为 0 ，可以看成任意数字。A 不能视为 14。

 

示例 1:

```
输入: [1,2,3,4,5]
输出: True
```


示例 2:

```
输入: [0,0,1,2,5]
输出: True
```


限制：

- 数组长度为 5 

- 数组的数取值为 [0, 13]

---

此 5 张牌是顺子的 充分条件 如下：

- 除大小王外，所有牌 无重复 ； 

- 设此 5 张牌中最大的牌为 max，最小的牌为 min （大小王除外），则需满足：max−min<5 

![Picture1.png](https://pic.leetcode-cn.com/df03847e2d04a3fcb5649541d4b6733fb2cb0d9293c3433823e04935826c33ef-Picture1.png)

方法一： 集合 Set + 遍历

遍历五张牌，遇到大小王（即 0 ）直接跳过。

判别重复： 利用 Set 实现遍历判重， Set 的查找方法的时间复杂度为 O(1) ；

获取最大 / 最小的牌： 借助辅助变量 ma 和 mi，遍历统计即可。 

```java
class Solution {
    public boolean isStraight(int[] nums) {
        Set<Integer> repeat = new HashSet<>();
        int max = 0, min = 14;
        for(int num : nums) {
            if(num == 0) continue; // 跳过大小王
            max = Math.max(max, num); // 最大牌
            min = Math.min(min, num); // 最小牌
            if(repeat.contains(num)) return false; // 若有重复，提前返回 false
            repeat.add(num); // 添加此牌至 Set
        }
        return max - min < 5; // 最大牌 - 最小牌 < 5 则可构成顺子
    }
}
```

- 时间复杂度 O(N) = O(5) = O(1) ： 其中 NN 为 nums 长度，本题中 N≡5 ；遍历数组使用 O(N) 时间。
- 空间复杂度 O(N) = O(5) = O(1)： 用于判重的辅助 Set 使用 O(N) 额外空间。

方法二：排序 + 遍历

先对数组执行排序。

判别重复： 排序数组中的相同元素位置相邻，因此可通过遍历数组，判断 `nums[i] = nums[i + 1]`是否成立来判重。

获取最大 / 最小的牌： 排序后，数组末位元素 `nums[4]` 为最大牌；元素 `nums[joker]`为最小牌，其中 joker 为大小王的数量。

```java
class Solution {
    public boolean isStraight(int[] nums) {
        int joker = 0;
        Arrays.sort(nums); // 数组排序
        for(int i = 0; i < 4; i++) {
            if(nums[i] == 0) joker++; // 统计大小王数量
            else if(nums[i] == nums[i + 1]) return false; // 若有重复，提前返回 false
        }
        return nums[4] - nums[joker] < 5; // 最大牌 - 最小牌 < 5 则可构成顺子
    }
}
```

- 时间复杂度 O(NlogN)=O(5log5)=O(1) ： 其中 N 为 nums长度，本题中 N≡5 ；数组排序使用 O(NlogN) 时间。
- 空间复杂度 O(1) ： 变量 joker 使用 O(1) 大小的额外空间。 

### [034] 顺时针打印矩阵

输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字。 

示例 1：

```
输入：matrix = [[1,2,3],[4,5,6],[7,8,9]]
输出：[1,2,3,6,9,8,7,4,5]
```

示例 2：

```
输入：matrix = [[1,2,3,4],[5,6,7,8],[9,10,11,12]]
输出：[1,2,3,4,8,12,11,10,9,5,6,7]
```


限制：

- 0 <= matrix.length <= 100
- 0 <= matrix[i].length <= 100 



方法一：

![Picture1.png](https://pic.leetcode-cn.com/c6de3a1bc0f38820941dbcff0e17a49204eba91b967d4ccc0d5485e68a4fcc95-Picture1.png)

**空值处理：** 当 `matrix` 为空时，直接返回空列表 `[]` 即可

**初始化：** 矩阵 左、右、上、下 四个边界 `l` , `r` , `t` , `b` ，用于打印的结果列表 `res` 。

**循环打印：** “从左向右、从上向下、从右向左、从下向上” 四个方向循环，每个方向打印中做以下三件事

1. 根据边界打印，即将元素按顺序添加至列表 `res` 尾部；
2. 边界向内收缩 1 （代表已被打印）；
3. 判断是否打印完毕（边界是否相遇），若打印完毕则跳出。

**返回值：** 返回 `res` 即可。

```java
class Solution {
    public int[] spiralOrder(int[][] matrix) {
        if(matrix.length == 0) return new int[0];
        int l = 0, r = matrix[0].length - 1, t = 0, b = matrix.length - 1, x = 0;
        int[] res = new int[(r + 1) * (b + 1)];
        while(true) {
            for(int i = l; i <= r; i++) res[x++] = matrix[t][i]; // left to right.
            if(++t > b) break;
            for(int i = t; i <= b; i++) res[x++] = matrix[i][r]; // top to bottom.
            if(l > --r) break;
            for(int i = r; i >= l; i--) res[x++] = matrix[b][i]; // right to left.
            if(t > --b) break;
            for(int i = b; i >= t; i--) res[x++] = matrix[i][l]; // bottom to top.
            if(++l > r) break;
        }
        return res;
    }
}
```

- 时间复杂度 O(MN) ： M, N 分别为矩阵行数和列数。
- 空间复杂度 O(1) ： 四个边界 l , r , t , b 使用常数大小的 额外 空间（ res 为必须使用的空间）。

### [035] 滑动窗口的最大值

给定一个数组 nums 和滑动窗口的大小 k，请找出所有滑动窗口里的最大值。

示例:

```
输入: nums = [1,3,-1,-3,5,3,6,7], 和 k = 3
输出: [3,3,5,5,6,7] 
解释: 

  滑动窗口的位置                最大值

---------------               -----

[1  3  -1] -3  5  3  6  7       3
 1 [3  -1  -3] 5  3  6  7       3
 1  3 [-1  -3  5] 3  6  7       5
 1  3  -1 [-3  5  3] 6  7       5
 1  3  -1  -3 [5  3  6] 7       6
 1  3  -1  -3  5 [3  6  7]      7
```


提示：

你可以假设 k 总是有效的，在输入数组不为空的情况下，1 ≤ k ≤ 输入数组的大小。

方法一：

[题解](https://leetcode-cn.com/problems/hua-dong-chuang-kou-de-zui-da-zhi-lcof/solution/mian-shi-ti-59-i-hua-dong-chuang-kou-de-zui-da-1-6/)

```java
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        if(nums.length == 0 || k == 0) return new int[0];
        Deque<Integer> deque = new LinkedList<>();
        int[] res = new int[nums.length - k + 1];
        for(int i = 0; i < k; i++) { // 未形成窗口
            while(!deque.isEmpty() && deque.peekLast() < nums[i])
                deque.removeLast();
            deque.addLast(nums[i]);
        }
        res[0] = deque.peekFirst();
        for(int i = k; i < nums.length; i++) { // 形成窗口后
            if(deque.peekFirst() == nums[i - k])
                deque.removeFirst();
            while(!deque.isEmpty() && deque.peekLast() < nums[i])
                deque.removeLast();
            deque.addLast(nums[i]);
            res[i - k + 1] = deque.peekFirst();
        }
        return res;
    }
}
```

- 时间复杂度 O(n)： 其中 n 为数组 nums 长度；线性遍历 nums 占用 O(N) ；每个元素最多仅入队和出队一次，因此单调队列 deque 占用 O(2N) 。
- 空间复杂度 O(k)： 双端队列 deque 中最多同时存储 k 个元素（即窗口大小）。

### [036] 0～n-1中缺失的数字 

一个长度为n-1的递增排序数组中的所有数字都是唯一的，并且每个数字都在范围0～n-1之内。在范围0～n-1内的n个数字中有且只有一个数字不在该数组中，请找出这个数字。

示例 1:

```
输入: [0,1,3]
输出: 2
```

示例 2:

```
输入: [0,1,2,3,4,5,6,7,9]
输出: 8
```


限制：

- 1 <= 数组长度 <= 10000

方法一：

根据题意，数组可以按照以下规则划分为两部分。

- 左子数组： nums[i] = i
- 右子数组： nums[i] != i

缺失的数字等于 “右子数组的首位元素” 对应的索引；因此考虑使用二分法查找 “右子数组的首位元素”

![Picture1.png](https://pic.leetcode-cn.com/df7e04fbab0937ff74e5f29e958c7b1d531af066789ff363be5e1c8e75f17f56-Picture1.png)



```java
class Solution {
    public int missingNumber(int[] nums) {
        int i = 0, j = nums.length - 1;
        while(i <= j) {
            int m = (i + j) / 2;
            if(nums[m] == m) i = m + 1;
            else j = m - 1;
        }
        return i;
    }
}
```

- 时间复杂度 O(log N) 
- 空间复杂度 O(1)

### [037] 翻转单词顺序 

输入一个英文句子，翻转句子中单词的顺序，但单词内字符的顺序不变。为简单起见，标点符号和普通字母一样处理。例如输入字符串"I am a student. "，则输出"student. a am I"。 

示例 1：

```
输入: "the sky is blue"
输出: "blue is sky the"
```

示例 2：

```
输入: "  hello world!  "
输出: "world! hello"
解释: 输入字符串可以在前面或者后面包含多余的空格，但是反转后的字符不能包括。
```

示例 3：

```
输入: "a good   example"
输出: "example good a"
解释: 如果两个单词间有多余的空格，将反转后单词间的空格减少到只含一个。
```

说明：

- 无空格字符构成一个单词。
- 输入字符串可以在前面或者后面包含多余的空格，但是反转后的字符不能包括。
- 如果两个单词间有多余的空格，将反转后单词间的空格减少到只含一个。

方法一：双指针

倒序遍历字符串 s ，记录单词左右索引边界 i , j ；每确定一个单词的边界，则将其添加至单词列表 res ；最终，将单词列表拼接为字符串，并返回即可。

```java
class Solution {
    public String reverseWords(String s) {
        s = s.trim(); // 删除首尾空格
        int j = s.length() - 1, i = j;
        StringBuilder res = new StringBuilder();
        while(i >= 0) {
            while(i >= 0 && s.charAt(i) != ' ') i--; // 搜索首个空格
            res.append(s.substring(i + 1, j + 1) + " "); // 添加单词
            while(i >= 0 && s.charAt(i) == ' ') i--; // 跳过单词间空格
            j = i; // j 指向下个单词的尾字符
        }
        return res.toString().trim(); // 转化为字符串并返回
    }
}
```

- 时间复杂度 O(N)
- 空间复杂度 O(N)

方法二：分割 + 倒序

利用 “字符串分割”、“列表倒序” 的内置函数，可简便地实现本题的字符串翻转要求。

```java
class Solution {
    public String reverseWords(String s) {
        String[] strs = s.trim().split(" "); // 删除首尾空格，分割字符串
        StringBuilder res = new StringBuilder();
        for(int i = strs.length - 1; i >= 0; i--) { // 倒序遍历单词列表
            if(strs[i].equals("")) continue; // 遇到空单词则跳过
            res.append(strs[i] + " "); // 将单词拼接至 StringBuilder
        }
        return res.toString().trim(); // 转化为字符串，删除尾部空格，并返回
    }
}
```

- 时间复杂度 O(N)
- 空间复杂度 O(N)



### [038] 青蛙跳台阶问题

一只青蛙一次可以跳上1级台阶，也可以跳上2级台阶。求该青蛙跳上一个 n 级的台阶总共有多少种跳法。

答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

示例 1：

```
输入：n = 2
输出：2
```

示例 2：

```
输入：n = 7
输出：21
```

示例 3：

```
输入：n = 0
输出：1
```

提示：

- 0 <= n <= 100

方法一：

设跳上 nn 级台阶有 f(n) 种跳法。在所有跳法中，青蛙的最后一步只有两种情况： 跳上 1 级或 2 级台阶。

- 当为 1 级台阶： 剩 n-1 个台阶，此情况共有 f(n−1) 种跳法；
- 当为 2 级台阶： 剩 n-2 个台阶，此情况共有 f(n−2) 种跳法。

f(n) 为以上两种情况之和，即 f(n)=f(n-1)+f(n-2)，以上递推性质为斐波那契数列。

> 斐波那契数列的定义是 f(n + 1) = f(n) + f(n - 1) ，生成第 n 项的做法有以下几种：
>
> 递归法：
> 原理： 把 f(n) 问题的计算拆分成 f(n-1)和 f(n-2) 两个子问题的计算，并递归，以 f(0) 和 f(1)为终止条件。
> 缺点： 大量重复的递归计算，例如 f(n)和 f(n - 1)两者向下递归都需要计算 f(n - 2)的值。
> 记忆化递归法：
> 原理： 在递归法的基础上，新建一个长度为 n 的数组，用于在递归时存储 f(0) 至 f(n) 的数字值，重复遇到某数字时则直接从数组取用，避免了重复的递归计算。
> 缺点： 记忆化存储的数组需要使用 O(N) 的额外空间。
> 动态规划：
> 原理： 以斐波那契数列性质 f(n + 1) = f(n) + f(n - 1)为转移方程。
> 从计算效率、空间复杂度上看，动态规划是本题的最佳解法。

动态规划解析：

状态定义： 设 dp 为一维数组，其中 dp[i] 的值代表 斐波那契数列第 i 个数字 。
转移方程： dp[i + 1] = dp[i] + dp[i - 1]，即对应数列定义 f(n + 1) = f(n) + f(n - 1)；
初始状态： dp[0] = 1dp[0]=1，即初始化前两个数字；
返回值： dp[n]，即斐波那契数列的第 n 个数字。

空间复杂度优化：

若新建长度为 n 的 dp 列表，则空间复杂度为 O(N)。

由于 dp 列表第 i 项只与第 i-1 和第 i-2 项有关，因此只需要初始化三个整形变量 sum, a, b ，利用辅助变量 sum 使 a, b 两数字交替前进即可 。
因为节省了 dp 列表空间，因此空间复杂度降至 O(1) 。

```java
class Solution {
    public int numWays(int n) {
        int a = 1, b = 1, sum;
        for(int i = 0; i < n; i++){
            sum = (a + b) % 1000000007;
            a = b;
            b = sum;
        }
        return a;
    }
}
```

- 时间复杂度 O(N)： 计算 f(n) 需循环 n 次，每轮循环内计算操作使用 O(1)。
- 空间复杂度 O(1) ： 几个标志变量使用常数大小的额外空间。



### [039] 波那契数列  

写一个函数，输入 n ，求斐波那契（Fibonacci）数列的第 n 项。斐波那契数列的定义如下：

```
F(0) = 0,   F(1) = 1
F(N) = F(N - 1) + F(N - 2), 其中 N > 1.
```

斐波那契数列由 0 和 1 开始，之后的斐波那契数就是由之前的两数相加而得出。

答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

示例 1：

```
输入：n = 2
输出：1
```

示例 2：

```
输入：n = 5
输出：5
```


提示：

- 0 <= n <= 100

方法：

同上题038

```java
class Solution {
    public int fib(int n) {
        int a = 0, b = 1, sum;
        for(int i = 0; i < n; i++){
            sum = (a + b) % 1000000007;
            a = b;
            b = sum;
        }
        return a;
    }
}
```

- 时间复杂度 O(N) 
- 空间复杂度 O(1)  