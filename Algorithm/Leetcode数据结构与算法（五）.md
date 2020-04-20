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

### 