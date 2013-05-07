算法导论第15章习题答案
========================

:date: 2013-05-07 09:51:00
:tags: clrs, algorithm

**15.2-2**

Q:请给出一个递归算法MATRIX-CHAIN-MULTIPLY(A,s,i,j)，使之在给出矩阵序列<A1,A2,...,An>，和由MATRIX-CHAIN-ORDER计算出的表s，
以及下标i和j后，能得出一个最有的矩阵链乘法。(初始调用为MATRIX-CHAIN-MULTIPLY(A,s,1,n))。

A:模仿PRINT-OPTIMAL-PARENS(s,i,j)即可。伪代码:

.. code-block:: c

    MATRIX-CHAIN-MULTIPLY(A,s,i,j)
    {
        if i=j then return Ai;
        else
            return MATRIX-CHAIN-MULTIPLY(A,s,i,s[i,j])*MATRIX-CHAIN-MULTIPLY(A,s[i,j]+1,j);
    }


**15.4-5**

Q:请给出一个O(n^2)时间的算法，使之能找出一个n个数的序列中最长的单调递增子序列。

A:这里给出两种算法。

方法1:假设这n个数存在数组X中，首先将这n个数按单调递增的顺序排序，将排好序的数组存于Y中。
然后通过计算X和Y的最长公共子序列即可找到答案。

方法2:直接利用动态规划的思想。设f(i)为以第i个数结尾的最长单调递增子序列的长度，
则f(i)=max(f(k)+1, 其中k<i且X[k]<X[i])，如果在1到i-1内找不到符合条件的k，那么f(i)就只能等于1了。
为了方便最后构造最长子序列，还需要另一个数组pre[i]存储以第i个数结尾的最长递增子序列的倒数第2个数。
伪代码:

.. code-block:: c

    LIS-Length(X)
    {
        max = 0;
        max_i = 0;
        f[1..n] = 1;
        pre[1..n] = -1;

        for i = 1 to n do
        {    
            for k = 1 to i-1 do
                if X[k]<X[i] and f[k]+1 > f[i] then
                {
                    f[i] = f[k+1];
                    pre[i] = k;
                }
        }

        for i = 1 to n do
            if f[i] > max then 
                max = f[i];
                max_i = i
        return max;
    }

    LIS-print(i)
    {
        if pre[i] != -1 
            LIS-print(pre[i]);
        print i;
    }


**15.4-6**

Q:请给出一个O(nlgn)时间的算法，使之能找出一个n个数的序列中最长的单调递增子序列。
(提示:观察长度为i的一个候选因子序列的最后一个元素，它至少与长度为i-1的一个候选子序列的最后一个元素一样大。通过把候选子序列与输入序列相连接来维护它们)。

A:这个提示看了好几遍也没有看懂说的是什么意思，大概是翻译的问题吧。总之需要将算法改进到O(nlgn)的复杂度。
自己想不到好的办法，百度了一下，想法还是挺巧妙的，用一个数组D[len]记录len长的递增子序列的最小末尾元素。
只需要从1到n一遍扫描中间加上二分查找，可以得到O(nlgn)的算法。这里不详细讨论，附上一个连接:

`最长递增子序列 <http://blog.csdn.net/ssjhust123/article/details/7798737>`_

