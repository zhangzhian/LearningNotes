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

