# Leetcode数据结构与算法

### [0097] 一维数组的动态和

给你一个数组 nums 。数组「动态和」的计算公式为：runningSum[i] = sum(nums[0]…nums[i]) 。

请返回 nums 的动态和。

 示例 1：

```
输入：nums = [1,2,3,4]
输出：[1,3,6,10]
解释：动态和计算过程为 [1, 1+2, 1+2+3, 1+2+3+4] 。
```


示例 2：

```
输入：nums = [1,1,1,1,1]
输出：[1,2,3,4,5]
解释：动态和计算过程为 [1, 1+1, 1+1+1, 1+1+1+1, 1+1+1+1+1] 。
```


示例 3：

```
输入：nums = [3,1,2,10,1]
输出：[3,4,6,16,17]
```

提示：

```
1 <= nums.length <= 1000
-10^6 <= nums[i] <= 10^6
```

方法一：

```java
class Solution {
    public int[] runningSum(int[] nums) {
        int[] ret = new int[nums.length];
        for(int i= 0;i<nums.length;i++){
           ret[i] = (i == 0 ? nums[0] : nums[i]+ret[i-1]);
        }
        return ret;
    }
}
```

### [0098] 好数对的数目

给你一个整数数组 nums 。

如果一组数字 (i,j) 满足 nums[i] == nums[j] 且 i < j ，就可以认为这是一组 好数对 。

返回好数对的数目。

示例 1：

```
输入：nums = [1,2,3,1,1,3]
输出：4
解释：有 4 组好数对，分别是 (0,3), (0,4), (3,4), (2,5) ，下标从 0 开始
```


示例 2：

```
输入：nums = [1,1,1,1]
输出：6
解释：数组中的每组数字都是好数对
```


示例 3：

```
输入：nums = [1,2,3]
输出：0
```

提示：

```
1 <= nums.length <= 100
1 <= nums[i] <= 100
```

方法一：

```java
class Solution {
    public int numIdenticalPairs(int[] nums) {
        int ret = 0;
        for(int i=0;i<nums.length;i++)
            for(int j=i+1;j<nums.length;j++)
                if(nums[i] == nums[j]) ret++;
        return ret;
    }
}
```

- 时间复杂度：O(n^2))。
- 空间复杂度：O(1)。

方法二：
假如输入[1,1,1,1]。我们从前往后遍历的时候，好数对的数量为：

3: [0,1],[0,2],[0,3]

2: [1,2],[1,3]

1: [2,3]

这里我们举的例子是[1,1,1,1]，如果我们的例子是[2,1,1,1,1,4]结果也是一样的，所以当我们从前往后遍历的时候，我们把数值存放在temp数组中，实际上计算的结果顺序是从后往前的，即1 + 2 + 3 = 6。这个结果其实跟我们正常理解的3 + 2 + 1 = 6的结果是一致的。

```java
class Solution {
    public int numIdenticalPairs(int[] nums) {
        int[] temp = new int[101];
        int ans = 0;
        for(int num : nums){
            //temp[num]存放的就是满足nums[i] == nums[j]的数目
            temp[num]++;
            //temp[num]-1理解为：满足nums[i] == nums[j]的好数对的数目。只不过这里的求值过程为倒序的
            ans+=temp[num]-1;
        }
        return ans;
    }
}
```

```java
class Solution {
    public int numIdenticalPairs(int[] nums) {
        int[] temps = new int[101];
        int ans = 0;
        for(int num : nums){
            temps[num]++;
        }
        for(int temp : temps){
            if(temp != 0) ans += temp * (temp - 1) /2;
        }
        return an;
    }
}
```

- 时间复杂度：O(n)。
- 空间复杂度：O(n)。

### [0099] 数组异或操作

给你两个整数，n 和 start 。

数组 nums 定义为：nums[i] = start + 2*i（下标从 0 开始）且 n == nums.length 。

请返回 nums 中所有元素按位异或（XOR）后得到的结果。

 

示例 1：

```
输入：n = 5, start = 0
输出：8
解释：数组 nums 为 [0, 2, 4, 6, 8]，其中 (0 ^ 2 ^ 4 ^ 6 ^ 8) = 8 。
     "^" 为按位异或 XOR 运算符。
```

示例 2：

```
输入：n = 4, start = 3
输出：8
解释：数组 nums 为 [3, 5, 7, 9]，其中 (3 ^ 5 ^ 7 ^ 9) = 8.
```


示例 3：

```
输入：n = 1, start = 7
输出：7
```


示例 4：

```
输入：n = 10, start = 5
输出：2
```


提示：

```
1 <= n <= 1000
0 <= start <= 1000
n == nums.length
```

方法一：

```java
class Solution {
    public int xorOperation(int n, int start) {
        int ans = 0;
        
        for(int i = 0; i < n; i++) {
            int elem = start + 2 * i;
            ans = ans ^ elem;
        }
        
        return ans;
    }
}
```

方法二：位运算（利用 XOR 的特性）

https://leetcode-cn.com/problems/xor-operation-in-an-array/solution/o1-wei-yun-suan-by-bruceyuj/

```java
class Solution {
    public int xorOperation(int n, int start) {
        int ans = 2 * xor(n, (int)Math.floor(start/2));
        
        if((n & start & 1) != 0) ans++; // 处理最后一位
        
        return ans;
    }

    public int xor(int n, int start) { // 将公式转换成情况 1
        if((start & 1)!=0) return (start-1) ^ helper(n+1, start-1);
        else return helper(n, start);
    }
    
    public int helper(int n, int start) { // 情况 1
        if(n % 2 == 0) return (int)Math.floor(n/2) & 1;
        else return (int)Math.floor(n/2) & 1 ^ (start+n-1);
    }
}
```

### [0100] 判断能否形成等差数列

给你一个数字数组 arr 。

如果一个数列中，任意相邻两项的差总等于同一个常数，那么这个数列就称为 等差数列 。

如果可以重新排列数组形成等差数列，请返回 true ；否则，返回 false 。

 示例 1：

```
输入：arr = [3,5,1]
输出：true
解释：对数组重新排序得到 [1,3,5] 或者 [5,3,1] ，任意相邻两项的差分别为 2 或 -2 ，可以形成等差数列。
```

示例 2：

```
输入：arr = [1,2,4]
输出：false
解释：无法通过重新排序得到等差数列。
```

方法一：

```java
class Solution {
    public boolean canMakeArithmeticProgression(int[] arr) {
        Arrays.sort(arr);
        int d = arr[1] - arr[0];
        for (int i = 2; i < arr.length; i++) {
            if (arr[i] - arr[i - 1] != d) return false;
        }
        return true;
    }
}
```

### [0101] 换酒问题

小区便利店正在促销，用 numExchange 个空酒瓶可以兑换一瓶新酒。你购入了 numBottles 瓶酒。

如果喝掉了酒瓶中的酒，那么酒瓶就会变成空的。

请你计算 最多 能喝到多少瓶酒。

 示例 1：

**![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/07/19/sample_1_1875.png)**

```
输入：numBottles = 9, numExchange = 3
输出：13
解释：你可以用 3 个空酒瓶兑换 1 瓶酒。
所以最多能喝到 9 + 3 + 1 = 13 瓶酒。
```

示例 2：

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/07/19/sample_2_1875.png)

```
输入：numBottles = 15, numExchange = 4
输出：19
解释：你可以用 4 个空酒瓶兑换 1 瓶酒。
所以最多能喝到 15 + 3 + 1 = 19 瓶酒。
```


示例 3：

```
输入：numBottles = 5, numExchange = 5
输出：6
```

示例 4：

```
输入：numBottles = 2, numExchange = 3
输出：2
```


提示：

```
1 <= numBottles <= 100
2 <= numExchange <= 100
```

方法一：

```java
class Solution {
    public int numWaterBottles(int numBottles, int numExchange) {
        int ret = numBottles;
        int temp1 = numBottles/numExchange; 
        int temp2 = numBottles%numExchange; 
        int tmp = temp1 + temp2;
        while(tmp >= numExchange){
            ret += temp1;
                
            temp1 = tmp / numExchange; 
            temp2 = tmp % numExchange;
            tmp = temp1 + temp2; 
        }
        ret += temp1;    
        return ret;
    }
}
```

方法二：

```
class Solution {
    public int numWaterBottles(int numBottles, int numExchange) {
        int bottle = numBottles, ans = numBottles;
        while (bottle >= numExchange) {
            bottle -= numExchange;
            ++ans;
            ++bottle;
        }
        return ans;
    }
}
```

方法三：(数学解法)
https://leetcode-cn.com/problems/water-bottles/solution/huan-jiu-wen-ti-by-leetcode-solution/

```
class Solution {
    public int numWaterBottles(int numBottles, int numExchange) {
        return numBottles >= numExchange ? (numBottles - numExchange) / (numExchange - 1) + 1 + numBottles : numBottles;
    }
}
```





























