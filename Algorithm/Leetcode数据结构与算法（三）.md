# Leetcode数据结构与算法

###[0033]从尾到头打印链表

输入一个链表的头节点，从尾到头反过来返回每个节点的值（用数组返回）。

**示例 1：**

```
输入：head = [1,3,2]
输出：[2,3,1]
```

**限制：**

```
0 <= 链表长度 <= 10000
```

方法一： 栈

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
    public int[] reversePrint(ListNode head) {
        Stack<Integer> stack=new Stack();

        while(head != null){
            stack.push(head.val);
            head = head.next;
        }
        
        int[] res = new int[stack.size()];
        int pos = 0;
        while(!stack.empty())
            res[pos++] = stack.pop();
        return res;
    }
}
```

时间复杂度 O(N)： 入栈和出栈共使用 O(N) 时间。
空间复杂度 O(N)： 辅助栈 stack 和数组 res 共使用 O(N) 的额外空间。

方法二：递归

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
    int i;//全局变量
    public int[] reversePrint(ListNode head) {
        recur(head,0);//递归函数调用
        return res;
    }
    void recur(ListNode head,int count) {
        if(head == null)
        {//递归终止条件
            res = new int[count];
            i = count-1;
            return;
        }
        recur(head.next,count+1);
        res[i-count] =  head.val; //这里的i-count是重点
    }
}
```

时间复杂度 O(N)： 遍历链表，递归 N 次。
空间复杂度 O(N)： 系统递归需要使用 O(N) 的栈空间。

###[0034]反转链表

定义一个函数，输入一个链表的头节点，反转该链表并输出反转后链表的头节点。

 示例:

```
输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL
```

**限制：**

```
0 <= 节点个数 <= 5000
```

**注意**：本题与0002 题相同

**方法一：迭代**

假设存在链表 1 → 2 → 3 → Ø，我们想要把它改成 Ø ← 1 ← 2 ← 3。

在遍历列表时，将当前节点的 next 指针改为指向前一个元素。由于节点没有引用其上一个节点，因此必须事先存储其前一个元素。在更改引用之前，还需要另一个指针来存储下一个节点。不要忘记在最后返回新的头引用！

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
public ListNode reverseList(ListNode head) {
    //申请节点，pre和 cur，pre指向null
    ListNode pre = null;
    ListNode cur = head;
  	ListNode tmp = null;
    while (cur != null) {
        tmp = cur.next;//记录当前节点的下一个节点
        cur.next = pre;//然后将当前节点指向pre
        //pre和cur节点都前进一位
        pre = cur;
        cur = tmp;
    }
    return pre;
}
```

**复杂度分析**

- 时间复杂度：O(n)，假设 n*n* 是列表的长度，时间复杂度是 O(n)。
- 空间复杂度：O(1)。

**方法二：递归**

递归的两个条件：

1. 终止条件是当前节点或者下一个节点==null
2. 在函数内部，改变节点的指向，也就是 head 的下一个节点指向 head 递归函数那句

```java
head.next.next = head
```

很不好理解，其实就是 head 的下一个节点指向head。
递归函数中每次返回的 cur 其实只最后一个节点，在递归函数内部，改变的是当前节点的指向。

```java
public ListNode reverseList(ListNode head) {
    if (head == null || head.next == null) return head;
    ListNode p = reverseList(head.next);
    head.next.next = head;
    head.next = null;
    return p;
}
```

**复杂度分析**

- 时间复杂度：O(n)，假设 n 是列表的长度，那么时间复杂度为O(n)。
- 空间复杂度：O(n)，由于使用递归，将会使用隐式栈空间。递归深度可能会达到 n 层。

###[0035]奇数值单元格的数目

给你一个 n 行 m 列的矩阵，最开始的时候，每个单元格中的值都是 0。

另有一个索引数组 indices，indices[i] = [ri, ci] 中的 ri 和 ci 分别表示指定的行和列（从 0 开始编号）。

你需要将每对 [ri, ci] 指定的行和列上的所有单元格的值加 1。

请你在执行完所有 indices 指定的增量操作后，返回矩阵中 「奇数值单元格」 的数目。

示例 1：

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2019/11/06/e1.png)

```
输入：n = 2, m = 3, indices = [[0,1],[1,1]]
输出：6
解释：最开始的矩阵是 [[0,0,0],[0,0,0]]。
第一次增量操作后得到 [[1,2,1],[0,1,0]]。
最后的矩阵是 [[1,3,1],[1,3,1]]，里面有 6 个奇数。
```


示例 2：

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2019/11/06/e2.png)

```
输入：n = 2, m = 2, indices = [[1,1],[0,0]]
输出：0
解释：最后的矩阵是 [[2,2],[2,2]]，里面没有奇数。
```

提示：

```
1 <= n <= 50
1 <= m <= 50
1 <= indices.length <= 100
0 <= indices[i][0] < n
0 <= indices[i][1] < m
```



方法一：模拟

```java
class Solution {
    public int oddCells(int n, int m, int[][] indices) {
        int[][] ret = new int[n][m];
        int val = 0;
      	//遍历indices，修改返回数组值
        for(int i = 0;i<indices.length;i++){     
            for (int c = 0; c< m;c++)
                    ret[indices[i][0]][c]++;
            for (int c = 0; c< n;c++)
                    ret[c][indices[i][1]]++;     
        }
        //统计奇数个数
        for(int i = 0;i<n;i++){
            for(int j = 0;j<m;j++){
                if(ret[i][j] % 2 == 1)
                    val++;
            }
        }
        return val;
    }
}
```

时间复杂度： O(L(M+N)+MN)，其中  L 是 indices 数组的长度。

空间复杂度：O(MN)。

方法二：模拟 + 空间优化

```java
class Solution {
    public int oddCells(int n, int m, int[][] indices) {
        int[] row=new int[n];
        int[] col=new int[m];
        //统计行数和列数出现的次数
        for(int i=0;i<indices.length;i++) {
        	row[indices[i][0]]++;
        	col[indices[i][1]]++;
        }
        //指定点的值为行数和列数出现次数之和
        int ans=0;
        for(int i=0;i<n;i++)
        	for(int j=0;j<m;j++) {
        		if((row[i]+col[j])%2>0)
        			ans++;
        	}
        return ans;
    }
}
```

时间复杂度：O(L+MN)，其中  L 是 indices 数组的长度。

空间复杂度：O(M+N)。

方法三：

```java
class Solution {
    public int oddCells(int n, int m, int[][] indices) {
        boolean[] r = new boolean[n];
        boolean[] c = new boolean[m];
        int i;
        for (i = 0; i < indices.length; i++) {
            r[indices[i][0]] = !r[indices[i][0]];
            c[indices[i][1]] = !c[indices[i][1]];
        }
        int rr = 0, cc = 0;
        for (i = 0; i < r.length; i++) {
            if(r[i])rr++;
        }
        for (i = 0; i < c.length; i++) {
            if(c[i])cc++;
        }
        return rr * m + cc * n - rr * cc * 2;
    
    }
}
```



 