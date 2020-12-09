# 剑指offer：中等部分

### [001] 求1+2+…+n

求 1+2+...+n ，要求不能使用乘除法、for、while、if、else、switch、case等关键字及条件判断语句（A?B:C）。

示例 1：

```
输入: n = 3
输出: 6
```

示例 2：

```
输入: n = 9
输出: 45
```


限制：

- 1 <= n <= 10000

方法一：平均计算

**问题：** 此计算必须使用 **乘除法** ，因此本方法不可取。

```java
public int sumNums(int n) {
    return (1 + n) * n / 2;
}
```

**思考：**`/ 2`可以用`>>`替代，`*n`可以用什么替代？

考虑 A 和 B 两数相乘的时候我们如何利用加法和位运算来模拟，其实就是将 B 二进制展开，如果 B 的二进制表示下第 i 位为 1，那么这一位对最后结果的贡献就是 `A∗(1<<i)` ，即 `A<<i`。我们遍历 B 二进制展开下的每一位，累加起来就是最后的答案，这个方法也被称作「俄罗斯农民乘法」

```c++
int quickMulti(int A, int B) {
    int ans = 0;
    for ( ; B; B >>= 1) {
        if (B & 1) {
            ans += A;
        }
        A <<= 1;
    }
    return ans;
}
```

上面的 C++ 实现里我们还是需要循环语句，替换循环语句需要自己手动展开，因为题目数据范围 n 为 `[1,10000]`，所以 n 二进制展开最多不会超过 14 位，我们手动展开 14 层代替循环

```java
class Solution {
    public int sumNums(int n) {
        int ans = 0, A = n, B = n + 1;
        boolean flag;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        return ans >> 1;
    }
}
```

方法二： 迭代

**问题**： 循环必须使用 **while 或 fo**r，因此本方法不可取，直接排除。

```java
public int sumNums(int n) {
    int res = 0;
    for(int i = 1; i <= n; i++)
        res += i;
    return res;
}
```

方法三： 递归（推介）

**问题：** 终止条件需要使用 if，因此本方法不可取。

```java
public int sumNums(int n) {
    if(n == 1) return 1;
    n += sumNums(n - 1);
    return n;
}
```

**思考：** 除了 if 和 switch 等判断语句外，是否有其他方法可用来终止递归？

**逻辑运算符的短路效应：**

常见的逻辑运算符有三种，即 “与 \&\& ”，“或 ||”，“非 ! ” ；而其有重要的短路效应，如下所示：

```java
if(A && B)  // 若 A 为 false ，则 B 的判断不会执行（即短路），直接判定 A && B 为 false

if(A || B) // 若 A 为 true ，则 B 的判断不会执行（即短路），直接判定 A || B 为 true
```

本题需要实现 “当 n = 1 时终止递归” 的需求，可通过短路效应实现。

```java
n > 1 && sumNums(n - 1) // 当 n = 1 时 n > 1 不成立 ，此时 “短路” ，终止后续递归
```

```java
class Solution {
    int res = 0;
    public int sumNums(int n) {
        boolean x = n > 1 && sumNums(n - 1) > 0;
        res += n;
        return res;
    }
}
```

复杂度分析：

- 时间复杂度 O(n) ： 计算 n + (n-1) + ... + 2 + 1 需要开启 n 个递归函数。
- 空间复杂度 O(n) ： 递归深度达到 n ，系统使用 O(n) 大小的额外空间。



































