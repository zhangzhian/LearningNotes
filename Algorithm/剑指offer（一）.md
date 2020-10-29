# 剑指offer

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

