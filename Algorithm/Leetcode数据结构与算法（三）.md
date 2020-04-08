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

###[0036]6 和 9 组成的最大数字

给你一个仅由数字 6 和 9 组成的正整数 `num`。

你最多只能翻转一位数字，将 6 变成 9，或者把 9 变成 6 。

请返回你可以得到的最大数字。

示例 1：

```
输入：num = 9669
输出：9969
解释：
改变第一位数字可以得到 6669 。
改变第二位数字可以得到 9969 。
改变第三位数字可以得到 9699 。
改变第四位数字可以得到 9666 。
其中最大的数字是 9969 。
```


示例 2：

```
输入：num = 9996
输出：9999
解释：将最后一位从 6 变到 9，其结果 9999 是最大的数。
```


示例 3：

```
输入：num = 9999
输出：9999
解释：无需改变就已经是最大的数字了。
```


提示：

```
1 <= num <= 10^4
num 每一位上的数字都是 6 或者 9 。
```

方法一：

```java
class Solution {
    public int maximum69Number (int num) {
        String s = num + "";
        s = s.replaceFirst("6", "9");
        return Integer.valueOf(s);
    }
}
```

方法二：

```java
class Solution {
    public int maximum69Number (int num) {
        StringBuilder sb = new StringBuilder(String.valueOf(num));
        int index = sb.indexOf("6");
        if (index != -1) {
            sb.setCharAt(index, '9');
        }
        return Math.max(num, Integer.parseInt(sb.toString()));
    }
}
```

方法三：

```java
class Solution {
    public int maximum69Number (int num) {
        int res = num;
        char[] chars = String.valueOf(num).toCharArray();
        for (int i = 0; i < chars.length; i++) {
            if (chars[i] == '6') {
                chars[i] = '9';
                res = Math.max(res, Integer.parseInt(String.valueOf(chars)));
                break;
            }
        }
        return res;
    }
}
```

###[0037]合并二叉树

给定两个二叉树，想象当你将它们中的一个覆盖到另一个上时，两个二叉树的一些节点便会重叠。

你需要将他们合并为一个新的二叉树。合并的规则是如果两个节点重叠，那么将他们的值相加作为节点合并后的新值，否则不为 NULL 的节点将直接作为新二叉树的节点。

示例 1:

输入: 

```
	Tree 1                     Tree 2                  
          1                         2                             
         / \                       / \                            
        3   2                     1   3                        
       /                           \   \                      
      5                             4   7                  

```

输出: 
合并后的树:

```


	     3
	    / \
	   4   5
	  / \   \ 
	 5   4   7
```


注意: 合并必须从两个树的根节点开始。

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

     public TreeNode mergeTrees(TreeNode t1, TreeNode t2) {
        if (t1 == null)
            return t2;
        if (t2 == null)
            return t1;
        t1.val += t2.val;
        t1.left = mergeTrees(t1.left, t2.left);
        t1.right = mergeTrees(t1.right, t2.right);
        return t1;
    }
}
```

复杂度分析

时间复杂度：O(N)，其中N 是两棵树中节点个数的较小值。

空间复杂度：O(N)，在最坏情况下，会递归 N 层，需要 O(N) 的栈空间。

方法二：

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

     public TreeNode mergeTrees(TreeNode t1, TreeNode t2) {
       if (t1 == null)
            return t2;
        Stack < TreeNode[] > stack = new Stack < > ();
        stack.push(new TreeNode[] {t1, t2});
        while (!stack.isEmpty()) {
            TreeNode[] t = stack.pop();
            if (t[0] == null || t[1] == null) {
                continue;
            }
            t[0].val += t[1].val;
            if (t[0].left == null) {
                t[0].left = t[1].left;
            } else {
                stack.push(new TreeNode[] {t[0].left, t[1].left});
            }
            if (t[0].right == null) {
                t[0].right = t[1].right;
            } else {
                stack.push(new TreeNode[] {t[0].right, t[1].right});
            }
        }
        return t1;
    }
}
 ```

复杂度分析
时间复杂度：O(N)，其中 N 是两棵树中节点个数的较小值。

空间复杂度：O(N)，在最坏情况下，栈中会存放 N 个节点。

###[0038]将每个元素替换为右侧最大元素

给你一个数组 arr ，请你将每个元素用它右边最大的元素替换，如果是最后一个元素，用 -1 替换。

完成所有替换操作后，请你返回这个数组。

 示例：

```
输入：arr = [17,18,5,4,6,1]
输出：[18,6,6,6,1,-1]
```

提示：

```
1 <= arr.length <= 10^4
1 <= arr[i] <= 10^5
```

方法一：暴力枚举

```java
class Solution {
    public int[] replaceElements(int[] arr) {
        int curMax = -1;
        int N = arr.length;
        for (int i = 0; i < N; i++) {
            int rightMax = -1;
            for (int j = i+1; j < N; j++) { 
                if(rightMax < arr[j])
                    rightMax = arr[j];  // 寻找当前位置i后面的最大值
        }
        arr[i] = rightMax; // 更新元素
        }
        arr[N-1] = -1;
        return arr;
    }
}
```

方法二：逆序遍历

```java
class Solution {
    public int[] replaceElements(int[] arr) {
        int rightMax = -1;
        for (int i = arr.length-1; i >= 0; i--) {
            int temp = arr[i];
            arr[i] = rightMax;
            if(temp > rightMax)
            rightMax = temp;
        }
        return arr;
    }
}
```



###[0038]汉明距离

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

###[0039]转换成小写字母

实现函数 ToLowerCase()，该函数接收一个字符串参数 str，并将该字符串中的大写字母转换成小写字母，之后返回新的字符串。 

示例 1：

```
输入: "Hello"
输出: "hello"
```


示例 2：

```
输入: "here"
输出: "here"
```


示例 3：

```
输入: "LOVELY"
输出: "lovely"
```

方法一：

```java
class Solution {
    public String toLowerCase(String str) {
        char[] arr = str.toCharArray();
        for (int i = 0; i < arr.length; i++) {
            if (arr[i] >= 'A' && arr[i] <= 'Z') {
                arr[i] += 'a' - 'A';
            }
        }
        return String.valueOf(arr);
    }
}
```

 方法二：

```java
class Solution {
    public String toLowerCase(String str) {
        StringBuilder s =  new StringBuilder();
        for(char c:str.toCharArray()){  //增强for（）循环
            if(c>=65 && c<=90)
                c+=32;
            s.append(c);
        }
        return s.toString();
    }
}
```

  方法三：

```java
class Solution {
    public String toLowerCase(String str) {
        return str.toLowerCase();     //也可以直接用函数实现
    }
}
```

###[0040]和为零的N个唯一整数

给你一个整数 `n`，请你返回 **任意** 一个由 `n` 个 **各不相同** 的整数组成的数组，并且这 `n` 个数相加和为 `0` 。

示例 1：

```
输入：n = 5
输出：[-7,-1,1,3,4]
解释：这些数组也是正确的 [-5,-1,1,2,3]，[-3,-1,2,-2,4]。
```


示例 2：

```
输入：n = 3
输出：[-1,0,1]
```


示例 3：

```
输入：n = 1
输出：[0]
```

提示：

```
1 <= n <= 1000
```

方法一：枚举

```java
public int[] sumZero(int n) {
    int sum = 0;
    int[] arr = new int[n];
    int len = 0;
    for (int i = 0; i <= n - 2; i++) {
        arr[len++] = i;
        sum += i;
    }
    arr[len] = -sum;
    return arr;
}
```

方法二：

```java
class Solution {
    public int[] sumZero(int n) {
        int[] arr = new int[n];
        search(1, n-1 , 1, arr);
        return arr;
    }

    private static void search(int l, int r, int val, int[] arr) {
        if (l > r)
            return;
        arr[l-1] = val;
        arr[r] = -val;
        search(l + 1, r - 1, val + 1, arr);
    }
}
```

```java
class Solution {
    public int[] sumZero(int n) {
        int[] arr = new int[n];
        int l = 0, r = n-1;     //左边界与右边界
        int i = 1;
        while (l < r) {
            arr[l++] = i;
            arr[r--] = -i;
            i++;
        }
        return arr;
    }
}
```

###[0041] 翻转二叉树

见[0025]二叉树的镜像  

###[0042]唯一摩尔斯密码词

国际摩尔斯密码定义一种标准编码方式，将每个字母对应于一个由一系列点和短线组成的字符串， 比如: "a" 对应 ".-", "b" 对应 "-...", "c" 对应 "-.-.", 等等。

为了方便，所有26个英文字母对应摩尔斯密码表如下：

```
[".-","-...","-.-.","-..",".","..-.","--.","....","..",".---","-.-",".-..","--","-.","---",".--.","--.-",".-.","...","-","..-","...-",".--","-..-","-.--","--.."]
```


给定一个单词列表，每个单词可以写成每个字母对应摩尔斯密码的组合。例如，"cab" 可以写成 "-.-..--..."，(即 "-.-." + "-..." + ".-"字符串的结合)。我们将这样一个连接过程称作单词翻译。

返回我们可以获得所有词不同单词翻译的数量。

例如:

```
输入: words = ["gin", "zen", "gig", "msg"]
输出: 2
解释: 
各单词翻译如下:
"gin" -> "--...-."
"zen" -> "--...-."
"gig" -> "--...--."
"msg" -> "--...--."

共有 2 种不同翻译, "--...-." 和 "--...--.".
```


注意:

- 单词列表words 的长度不会超过 100。
- 每个单词 words[i]的长度范围为 [1, 12]。
- 每个单词 words[i]只包含小写字母。

方法一：

```java
class Solution {
    public int uniqueMorseRepresentations(String[] words) {
        String[] MORSE = new String[]{".-","-...","-.-.","-..",".","..-.","--.",
                         "....","..",".---","-.-",".-..","--","-.",
                         "---",".--.","--.-",".-.","...","-","..-",
                         "...-",".--","-..-","-.--","--.."};

        Set<String> seen = new HashSet();
        for (String word: words) {
            StringBuilder code = new StringBuilder();
            for (char c: word.toCharArray())
                code.append(MORSE[c - 'a']);
            seen.add(code.toString());
        }

        return seen.size();
    }
}
```

###[0043]翻转图像

给定一个二进制矩阵 A，我们想先水平翻转图像，然后反转图像并返回结果。

水平翻转图片就是将图片的每一行都进行翻转，即逆序。例如，水平翻转 [1, 1, 0] 的结果是 [0, 1, 1]。

反转图片的意思是图片中的 0 全部被 1 替换， 1 全部被 0 替换。例如，反转 [0, 1, 1] 的结果是 [1, 0, 0]。

示例 1:

```
输入: [[1,1,0],[1,0,1],[0,0,0]]
输出: [[1,0,0],[0,1,0],[1,1,1]]
解释: 首先翻转每一行: [[0,1,1],[1,0,1],[0,0,0]]；
     然后反转图片: [[1,0,0],[0,1,0],[1,1,1]]
```


示例 2:

```
输入: [[1,1,0,0],[1,0,0,1],[0,1,1,1],[1,0,1,0]]
输出: [[1,1,0,0],[0,1,1,0],[0,0,0,1],[1,0,1,0]]
解释: 首先翻转每一行: [[0,0,1,1],[1,0,0,1],[1,1,1,0],[0,1,0,1]]；
     然后反转图片: [[1,1,0,0],[0,1,1,0],[0,0,0,1],[1,0,1,0]]
```


说明:

```
1 <= A.length = A[0].length <= 20
0 <= A[i][j] <= 1
```

 方法一：

```java
class Solution {
    public int[][] flipAndInvertImage(int[][] A) {
        int C = A[0].length;
        for (int[] row: A)
            for (int i = 0; i < (C + 1) / 2; ++i) {
                int tmp = row[i] ^ 1;
                row[i] = row[C - 1 - i] ^ 1;
                row[C - 1 - i] = tmp;
            }

        return A;
    }
}
```

###[0044]上升下降字符串

给你一个字符串 s ，请你根据下面的算法重新构造字符串：

从 s 中选出 最小 的字符，将它 接在 结果字符串的后面。
从 s 剩余字符中选出 最小 的字符，且该字符比上一个添加的字符大，将它 接在 结果字符串后面。
重复步骤 2 ，直到你没法从 s 中选择字符。
从 s 中选出 最大 的字符，将它 接在 结果字符串的后面。
从 s 剩余字符中选出 最大 的字符，且该字符比上一个添加的字符小，将它 接在 结果字符串后面。
重复步骤 5 ，直到你没法从 s 中选择字符。
重复步骤 1 到 6 ，直到 s 中所有字符都已经被选过。
在任何一步中，如果最小或者最大字符不止一个 ，你可以选择其中任意一个，并将其添加到结果字符串。

请你返回将 s 中字符重新排序后的 结果字符串 。

示例 1：

```
输入：s = "aaaabbbbcccc"
输出："abccbaabccba"
解释：第一轮的步骤 1，2，3 后，结果字符串为 result = "abc"
第一轮的步骤 4，5，6 后，结果字符串为 result = "abccba"
第一轮结束，现在 s = "aabbcc" ，我们再次回到步骤 1
第二轮的步骤 1，2，3 后，结果字符串为 result = "abccbaabc"
第二轮的步骤 4，5，6 后，结果字符串为 result = "abccbaabccba"
```


示例 2：

```
输入：s = "rat"
输出："art"
解释：单词 "rat" 在上述算法重排序以后变成 "art"
```


示例 3：

```
输入：s = "leetcode"
输出："cdelotee"
```


示例 4：

```
输入：s = "ggggggg"
输出："ggggggg"
```


示例 5：

```
输入：s = "spo"
输出："ops"
```

提示：

```
1 <= s.length <= 500
s 只包含小写英文字母。
```

方法一：

```java
class Solution {
    public String sortString(String s) {
        StringBuilder builder = new StringBuilder();
        int[] map = new int[26];
        for (char c : s.toCharArray()) map[c - 'a']++;
        boolean flag;
        do {
            flag = false;
            for (int i = 0; i < 26; i++) {
                if (map[i] > 0) {
                    builder.append((char) (i + 'a'));
                    map[i]--;
                    flag = true;
                }
            }
            for (int i = 25; i >= 0; i--) {
                if (map[i] > 0) {
                    builder.append((char) (i + 'a'));
                    map[i]--;
                    flag = true;
                }
            }
        } while (flag);
        return builder.toString();
    }
}
```



###[0045]机器人能否返回原点

在二维平面上，有一个机器人从原点 (0, 0) 开始。给出它的移动顺序，判断这个机器人在完成移动后是否在 (0, 0) 处结束。

移动顺序由字符串表示。字符 move[i] 表示其第 i 次移动。机器人的有效动作有 R（右），L（左），U（上）和 D（下）。如果机器人在完成所有动作后返回原点，则返回 true。否则，返回 false。

注意：机器人“面朝”的方向无关紧要。 “R” 将始终使机器人向右移动一次，“L” 将始终向左移动等。此外，假设每次移动机器人的移动幅度相同。

 示例 1:

```
输入: "UD"
输出: true
解释：机器人向上移动一次，然后向下移动一次。所有动作都具有相同的幅度，因此它最终回到它开始的原点。因此，我们返回 true。
```


示例 2:

```
输入: "LL"
输出: false
解释：机器人向左移动两次。它最终位于原点的左侧，距原点有两次 “移动” 的距离。我们返回 false，因为它在移动结束时没有返回原点。
```

方法一：

```java
class Solution {
    public boolean judgeCircle(String moves) {
        int sum = 0;
        for (char c : moves.toCharArray()){
            if(c == 'R' ) sum+=1;
            if(c == 'L' ) sum-=1;
            if(c == 'U' ) sum+=2;
            if(c == 'D' ) sum-=2;
        }
        return sum == 0;
    }
}
```

方法二：

```java
class Solution {
    public boolean judgeCircle(String moves) {
        int x = 0, y = 0;
        for (char move: moves.toCharArray()) {
            if (move == 'U') y--;
            else if (move == 'D') y++;
            else if (move == 'L') x--;
            else if (move == 'R') x++;
        }
        return x == 0 && y == 0;
    }
}
```

###[0046]解码字母到整数映射

给你一个字符串 s，它由数字（'0' - '9'）和 '#' 组成。我们希望按下述规则将 s 映射为一些小写英文字符：

字符（'a' - 'i'）分别用（'1' - '9'）表示。
字符（'j' - 'z'）分别用（'10#' - '26#'）表示。 
返回映射之后形成的新字符串。

题目数据保证映射始终唯一。

 示例 1：

```
输入：s = "10#11#12"
输出："jkab"
解释："j" -> "10#" , "k" -> "11#" , "a" -> "1" , "b" -> "2".
```


示例 2：

```
输入：s = "1326#"
输出："acz"
```


示例 3：

```
输入：s = "25#"
输出："y"
```


示例 4：

```
输入：s = "12345678910#11#12#13#14#15#16#17#18#19#20#21#22#23#24#25#26#"
输出："abcdefghijklmnopqrstuvwxyz"
```

提示：

```
1 <= s.length <= 1000
s[i] 只包含数字（'0'-'9'）和 '#' 字符。
s 是映射始终存在的有效字符串。
```

方法一：

```java
class Solution {
    public String freqAlphabets(String s) {
         // 1 - 9 只有一位数， 10 - 26 有 10# - 26# 三位数
        StringBuilder sb = new StringBuilder();
        for(int i = 0; i < s.length(); ){
            if(i + 2 < s.length() && s.charAt(i + 2) == '#'){
                sb.append((char) ('a' + Integer.parseInt(s.substring(i, i + 2)) - 1));
                i += 3;
            }else{
                sb.append((char) ('a' + Integer.parseInt(s.substring(i, i + 1)) - 1));
                i++;
            }
        }
        return sb.toString();
    }
}
```

###[0047]合并两个排序的链表

输入两个递增排序的链表，合并这两个链表并使新链表中的节点仍然是递增排序的。

示例1：

```
输入：1->2->4, 1->3->4
输出：1->1->2->3->4->4
```


限制：

```
0 <= 链表长度 <= 1000
```

方法一：迭代

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
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        // maintain an unchanging reference to node ahead of the return node.
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

        // exactly one of l1 and l2 can be non-null at this point, so connect
        // the non-null list to the end of the merged list.
        prev.next = l1 == null ? l2 : l1;

        return prehead.next;
    }
}
```

时间复杂度：O(n+m)

空间复杂度：O(1)

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
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if (l1 == null) {
            return l2;
        }
        else if (l2 == null) {
            return l1;
        }
        else if (l1.val < l2.val) {
            l1.next = mergeTwoLists(l1.next, l2);
            return l1;
        }
        else {
            l2.next = mergeTwoLists(l1, l2.next);
            return l2;
        }
    }
}
```

时间复杂度：O(n+m)

空间复杂度：O(n+m)

###[0048]化栈为队

实现一个MyQueue类，该类用两个栈来实现一个队列。

示例：

```
MyQueue queue = new MyQueue();

queue.push(1);
queue.push(2);
queue.peek();  // 返回 1
queue.pop();   // 返回 1
queue.empty(); // 返回 false
```

说明：

```
你只能使用标准的栈操作 -- 也就是只有 push to top, peek/pop from top, size 和 is empty 操作是合法的。
你所使用的语言也许不支持栈。你可以使用 list 或者 deque（双端队列）来模拟一个栈，只要是标准的栈操作即可。
假设所有操作都是有效的 （例如，一个空的队列不会调用 pop 或者 peek 操作）。
```

方法一：

```java
class MyQueue {

        private Stack<Integer> numStack;
        private Stack<Integer> helpStack;

        /** Initialize your data structure here. */
        public MyQueue() {
            numStack = new Stack<>();
            helpStack = new Stack<>();
        }

        /** Push element x to the back of queue. */
        public void push(int x) {
            numStack.push(x);
        }

        /** Removes the element from in front of queue and returns that element. */
        public int pop() {
            peek();
            return helpStack.pop();
        }

        /** Get the front element. */
        public int peek() {
            if (helpStack.isEmpty()) {
                while (!numStack.isEmpty()) {
                    helpStack.push(numStack.pop());
                }
            }

            return helpStack.peek();
        }

        /** Returns whether the queue is empty. */
        public boolean empty() {
            return helpStack.isEmpty() && numStack.isEmpty();
        }
}

/**
 * Your MyQueue object will be instantiated and called as such:
 * MyQueue obj = new MyQueue();
 * obj.push(x);
 * int param_2 = obj.pop();
 * int param_3 = obj.peek();
 * boolean param_4 = obj.empty();
 */
```



