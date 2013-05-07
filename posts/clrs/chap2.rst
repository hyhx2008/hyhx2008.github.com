算法导论第2章习题答案
======================

:tags: clrs, algorithm
:date: 2013-04-18 17:00:00

**2.3-7**

Q:请给出一个运行时间为O(nlgn)的算法，使之能在一个由n个整数构成的集合S和另一个整数X时，判断出S中是否存在有两个其和等于X的元素。

A:首先将这n个数进行排序，然后依次对每个数查找是否存在另一个数与其的和为x，查找使用二分查找法。伪代码:

.. code-block:: c

    checkSums(s,x)  //s为存放n个整数的集合
    {
        sort(s);    //排序

        for i = 1 to n do
            if binarySearch(s,x-s[i]) then  //二分查找
                return true;

        return false;
    }

排序的复杂度为nlgn，二分查找的复杂度为lgn，执行n次就是nlgn，所以该算法的复杂度为O(nlgn)。

**2-4 逆序对**

Q:设A[1..n]是一个包含n个不同数的数组。如果在i<j的情况下，有A[i]>A[j]，则(i,j)就称为A中的一个逆序对(inversion)。
给出一个算法，它能用O(nlgn)的最坏情况运行时间，确定n个元素的任何排列中的逆序对的数目。

A:题目提示我们修改合并排序算法，即在合并排序合并的过程中记录逆序对的个数。伪代码:

.. code-block:: c
    
    int cnt = 0;            //全局变量，记录逆序对的个数

    mergeSort(A,p,r)        //合并排序
    {
        if p < r then
        {
            q = (p+r)/2;
            mergeSort(A,p,q);
            mergeSort(A,q+1,r);
            merge(A,p,r,q);
        }
    }

    merge(A,p,q,r)
    {
        n1 = q-p+1;
        n2 = r-q;
        creat arrays L[1..n1+1] and R[1..n2+1]
        for i = 1 to n1 do
            L[i] = A[p+i-1];
        for j = 1 to n2 do
            R[j] = A[q+j];
        L[n1+1] = -1;
        R[n2+1] = -1;
        i = 1;
        j = 1;
        for k = p to r do 
            if (L[i] > R[j])            //序号小的元素大于序号大的元素,出现逆序对    
            {
                A[k] = L[i];
                i = i + 1; 
                cnt = cnt + n1 - i + 1; //计入由L[i]产生的所有逆序对  
            }else
            {
                A[k] = R[j];
                j = j + 1;
            }
    }


