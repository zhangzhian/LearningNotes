# Leetcode数据结构与算法

### [0081]按奇偶排序数组

给定一个非负整数数组 A，返回一个数组，在该数组中， A 的所有偶数元素之后跟着所有奇数元素。

你可以返回满足此条件的任何数组作为答案。 示例：

```
输入：[3,1,2,4]
输出：[2,4,3,1]
输出 [4,2,3,1]，[2,4,1,3] 和 [4,2,1,3] 也会被接受。
```

提示：

```
1 <= A.length <= 5000
0 <= A[i] <= 5000
```

方法一：排序

```java
class Solution {
    public int[] sortArrayByParity(int[] A) {
        Integer[] B = new Integer[A.length];
        for (int t = 0; t < A.length; ++t)
            B[t] = A[t];

        Arrays.sort(B, (a, b) -> Integer.compare(a%2, b%2));

        for (int t = 0; t < A.length; ++t)
            A[t] = B[t];
        return A;

        /* Alternative:
        return Arrays.stream(A)
                     .boxed()
                     .sorted((a, b) -> Integer.compare(a%2, b%2))
                     .mapToInt(i -> i)
                     .toArray();
        */
    }
}
```

方法二：两次扫描

```java
class Solution {
    public int[] sortArrayByParity(int[] A) {
        int[] ans = new int[A.length];
        int t = 0;

        for (int i = 0; i < A.length; ++i)
            if (A[i] % 2 == 0)
                ans[t++] = A[i];

        for (int i = 0; i < A.length; ++i)
            if (A[i] % 2 == 1)
                ans[t++] = A[i];

        return ans;
    }
}
```

方法三：原地算法

维护两个指针 i 和 j，循环保证每刻小于 i 的变量都是偶数（也就是 A[k] % 2 == 0 当 k < i），所有大于 j 的都是奇数。

所以， 4 种情况针对 (A[i] % 2, A[j] % 2)：

如果是 (0, 1)，那么 i++ 并且 j--。
如果是 (1, 0)，那么交换两个元素，然后继续。
如果是 (0, 0)，那么说明 i 位置是正确的，只能 i++。
如果是 (1, 1)，那么说明 j 位置是正确的，只能 j--。
通过这 4 种情况，循环不变量得以维护，并且 j-i 不断变小。最终就可以得到奇偶有序的数组。

```java
class Solution {
    public int[] sortArrayByParity(int[] A) {
        int i = 0, j = A.length - 1;
        while (i < j) {
            if (A[i] % 2 > A[j] % 2) {
                int tmp = A[i];
                A[i] = A[j];
                A[j] = tmp;
            }

            if (A[i] % 2 == 0) i++;
            if (A[j] % 2 == 1) j--;
        }

        return A;
    }
}
```



### [0082]重新排列数组

给你一个数组 nums ，数组中有 2n 个元素，按 [x1,x2,...,xn,y1,y2,...,yn] 的格式排列。

请你将数组按 [x1,y1,x2,y2,...,xn,yn] 格式重新排列，返回重排后的数组。

示例 1：

```
输入：nums = [2,5,1,3,4,7], n = 3
输出：[2,3,5,4,1,7] 
解释：由于 x1=2, x2=5, x3=1, y1=3, y2=4, y3=7 ，所以答案为 [2,3,5,4,1,7]
```


示例 2：

```
输入：nums = [1,2,3,4,4,3,2,1], n = 4
输出：[1,4,2,3,3,2,4,1]
```


示例 3：

```
输入：nums = [1,1,2,2], n = 2
输出：[1,2,1,2]
```

提示：

```
1 <= n <= 500
nums.length == 2n
1 <= nums[i] <= 10^3
```

方法：

```java
class Solution {
    public int[] shuffle(int[] nums, int n) {
        int[] ret = new int[2 * n];
        for(int i = 0; i < n; i++){
            ret[2 * i] = nums[i];
            ret[2 * i+1] = nums[i + n];
        }
        return ret;
    }
}
```



### [0083]拥有最多糖果的孩子

给你一个数组 candies 和一个整数 extraCandies ，其中 candies[i] 代表第 i 个孩子拥有的糖果数目。

对每一个孩子，检查是否存在一种方案，将额外的 extraCandies 个糖果分配给孩子们之后，此孩子有 最多 的糖果。注意，允许有多个孩子同时拥有 最多 的糖果数目。

 

示例 1：

```
输入：candies = [2,3,5,1,3], extraCandies = 3
输出：[true,true,true,false,true] 
```


解释：

```
孩子 1 有 2 个糖果，如果他得到所有额外的糖果（3个），那么他总共有 5 个糖果，他将成为拥有最多糖果的孩子。
孩子 2 有 3 个糖果，如果他得到至少 2 个额外糖果，那么他将成为拥有最多糖果的孩子。
孩子 3 有 5 个糖果，他已经是拥有最多糖果的孩子。
孩子 4 有 1 个糖果，即使他得到所有额外的糖果，他也只有 4 个糖果，无法成为拥有糖果最多的孩子。
孩子 5 有 3 个糖果，如果他得到至少 2 个额外糖果，那么他将成为拥有最多糖果的孩子。
```


示例 2：

```
输入：candies = [4,2,1,1,2], extraCandies = 1
输出：[true,false,false,false,false] 
解释：只有 1 个额外糖果，所以不管额外糖果给谁，只有孩子 1 可以成为拥有糖果最多的孩子。
```


示例 3：

```
输入：candies = [12,1,12], extraCandies = 10
输出：[true,false,true]
```

方法一：

```java
class Solution {
    public List<Boolean> kidsWithCandies(int[] candies, int extraCandies) {
        List<Boolean> list = new ArrayList<Boolean>();
        int maxValue = 0;
        for(int i = 0; i < candies.length ; i++ )
            if(candies[i] > maxValue) maxValue = candies[i];
        for(int i = 0; i < candies.length ; i++ )
            list.add(candies[i] +extraCandies >= maxValue ? true : false);
        return list;
    }
}
```



### [0084]在既定时间做作业的学生人数

给你两个整数数组 startTime（开始时间）和 endTime（结束时间），并指定一个整数 queryTime 作为查询时间。

已知，第 i 名学生在 startTime[i] 时开始写作业并于 endTime[i] 时完成作业。

请返回在查询时间 queryTime 时正在做作业的学生人数。形式上，返回能够使 queryTime 处于区间 [startTime[i], endTime[i]]（含）的学生人数。

 

示例 1：

```
输入：startTime = [1,2,3], endTime = [3,2,7], queryTime = 4
输出：1
解释：一共有 3 名学生。
第一名学生在时间 1 开始写作业，并于时间 3 完成作业，在时间 4 没有处于做作业的状态。
第二名学生在时间 2 开始写作业，并于时间 2 完成作业，在时间 4 没有处于做作业的状态。
第二名学生在时间 3 开始写作业，预计于时间 7 完成作业，这是是唯一一名在时间 4 时正在做作业的学生。
```


示例 2：

```
输入：startTime = [4], endTime = [4], queryTime = 4
输出：1
解释：在查询时间只有一名学生在做作业。
```


示例 3：

```
输入：startTime = [4], endTime = [4], queryTime = 5
输出：0
```


示例 4：

```
输入：startTime = [1,1,1,1], endTime = [1,3,2,4], queryTime = 7
输出：0
```


示例 5：

```
输入：startTime = [9,8,7,6,5,4,3,2,1], endTime = [10,10,10,10,10,10,10,10,10], queryTime = 5
输出：5
```

提示：

```
startTime.length == endTime.length
1 <= startTime.length <= 100
1 <= startTime[i] <= endTime[i] <= 1000
1 <= queryTime <= 1000
```

方法一：

```java
class Solution {
    public int busyStudent(int[] startTime, int[] endTime, int queryTime) {
        int ret = 0;
        for(int i = 0; i < startTime.length ; i++ )
            if(startTime[i] <= queryTime && endTime[i] >= queryTime)
                ret++;
        return ret;
    }
}
```

### [0085]拿硬币

桌上有 n 堆力扣币，每堆的数量保存在数组 coins 中。我们每次可以选择任意一堆，拿走其中的一枚或者两枚，求拿完所有力扣币的最少次数。

示例 1：

```
输入：[4,2,1]

输出：4

解释：第一堆力扣币最少需要拿 2 次，第二堆最少需要拿 1 次，第三堆最少需要拿 1 次，总共 4 次即可拿完。
```

示例 2：

```
输入：[2,3,10]

输出：8
```

限制：

```
1 <= n <= 4
1 <= coins[i] <= 10
```

方法一：

```java
class Solution {
    public int minCount(int[] coins) {
        int ret = 0;
        for(int i = 0; i < coins.length ; i++ ){
            ret += coins[i] % 2 == 0 ? coins[i] /2 : coins[i] /2 + 1 ;
        }
        return ret;
    }
}
```

### [0086]旅行终点站

给你一份旅游线路图，该线路图中的旅行线路用数组 paths 表示，其中 paths[i] = [cityAi, cityBi] 表示该线路将会从 cityAi 直接前往 cityBi 。请你找出这次旅行的终点站，即没有任何可以通往其他城市的线路的城市。

题目数据保证线路图会形成一条不存在循环的线路，因此只会有一个旅行终点站。

示例 1：

```
输入：paths = [["London","New York"],["New York","Lima"],["Lima","Sao Paulo"]]
输出："Sao Paulo" 
解释：从 "London" 出发，最后抵达终点站 "Sao Paulo" 。本次旅行的路线是 "London" -> "New York" -> "Lima" -> "Sao Paulo" 。
```


示例 2：

```
输入：paths = [["B","C"],["D","B"],["C","A"]]
输出："A"
解释：所有可能的线路是：
"D" -> "B" -> "C" -> "A". 
"B" -> "C" -> "A". 
"C" -> "A". 
"A". 
显然，旅行终点站是 "A" 。
```


示例 3：

```
输入：paths = [["A","Z"]]
输出："Z"
```

方法一：

```java
class Solution {
    public String destCity(List<List<String>> paths) {
		Map<String, Integer> map = new HashMap<String, Integer>();
		for (List<String> list : paths)
			map.put(list.get(0), 1);
		for (List<String> list : paths)
			if (map.get(list.get(1)) == null)
				return list.get(1);
		return null;
    }
}
```

### [0087]通过翻转子数组使两个数组相等

给你两个长度相同的整数数组 target 和 arr 。

每一步中，你可以选择 arr 的任意 非空子数组 并将它翻转。你可以执行此过程任意次。

如果你能让 arr 变得与 target 相同，返回 True；否则，返回 False 。

示例 1：

```
输入：target = [1,2,3,4], arr = [2,4,1,3]
输出：true
解释：你可以按照如下步骤使 arr 变成 target：
1- 翻转子数组 [2,4,1] ，arr 变成 [1,4,2,3]
2- 翻转子数组 [4,2] ，arr 变成 [1,2,4,3]
3- 翻转子数组 [4,3] ，arr 变成 [1,2,3,4]
上述方法并不是唯一的，还存在多种将 arr 变成 target 的方法。
```


示例 2：

```
输入：target = [7], arr = [7]
输出：true
解释：arr 不需要做任何翻转已经与 target 相等。
```


示例 3：

```
输入：target = [1,12], arr = [12,1]
输出：true
```


示例 4：

```
输入：target = [3,7,9], arr = [3,7,11]
输出：false
解释：arr 没有数字 9 ，所以无论如何也无法变成 target 。
```


示例 5：

```
输入：target = [1,1,1,1,1], arr = [1,1,1,1,1]
输出：true
```

提示：

```
target.length == arr.length
1 <= target.length <= 1000
1 <= target[i] <= 1000
1 <= arr[i] <= 1000
```

方法一：(实际上是比较两个数组是否相等，参考冒泡排序)

```java
class Solution {
    public boolean canBeEqual(int[] target, int[] arr) {
       if (target.length != arr.length) {
            return false;
        }
        int[] tmp1 = new int[1001];
        int[] tmp2 = new int[1001];
        int len = target.length;
        for(int i = 0; i < len ; i++)
        {   tmp1[target[i]]++;
            tmp2[arr[i]]++;
        }
        for(int i = 0; i < 1001 ; i++)
        {
            if(tmp1[i] != tmp2[i]) return false;
        }
        return true;
    }
}
```



