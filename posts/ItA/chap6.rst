算法导论第六章习题答案
======================

:date: 2013-04-19 17:20:00
:tags: 算法导论, 算法

**6.5-6**

Q:说明如何使用优先级队列来实现一个先进先出队列，另说明如何用优先级队列来实现栈。

A:队列的性质是先进先出，所以维护一个最小优先级队列，给先进队的元素赋一个小的优先级，每插入一个新的元素优先级加1。
出队时取优先级最小的元素并维护优先级队列即可。栈的实现同理。

**6.5-7**

Q:HEAP-DELETE(A,i)操作将结点i中的项从堆A中删去。对含n个元素的最大堆，请给出时间为O(lgn)的HEAP-DELETE的实现。

A:类似于堆排序时做的操作，将要删除的结点和堆的最后一个结点交换，将其删除后维护堆的性质。伪代码:

.. code-block:: c

    HEAP-DELETE(A,i)
    {
        A[i] = A[heap-size[A]];
        heap-size[A] = heap-size[A] - 1;
        key = A[i];
        if key <= A[PARENT[i]] then         //放在i位置的新元素应该在堆更下面的位置
            MAX-HEAPIFY(A,i);
        else                                //放在i位置的新元素应该在堆更上面的位置
            while i>1 and A[PARENT(i)] < key do
            {
                A[i] = A[PARENT[i]];
                i = PARENT(i);
            }
    }

**6-3 Young氏矩阵**

Q:一个 m x n 的Young氏矩阵(Young tableau)是一个 m x n 的矩阵，其中每一行的数据都从左到右排序，每一列的数据都从上到下排序。
Young氏矩阵中可能会有一些 ∞ 的数据项，表示不存在的元素。所以，Young氏矩阵可以用来存放 r <= mn个有限的数。请给出一个运行时间为O(m+n)的算法，
来决定一个给定的数是否存在于一个给定的 m x n 的Young氏矩阵内。

A:我们可以用给定的数和Young氏矩阵中最右上角的那个数比，如果给定的数比较大，则该数不可能存在于最上面那行；如果给定的数比较下，那么该数不可能存在于最右边那列。
这样每次比较就可以排除一行或者一列。重复此过程，直到找到给定的数或者所有的行和列都被排除了为止。伪代码:

.. code-block:: c

    Search-Young(A,i,j,x)
    {
        
        if x = A[i,j] then return true;
        if x > A[i,j] then
            if i < m then return Search-Young(A,i+1,j,x);
            else return false;
        if x < A[i,j] then
            if j > 1 then return Search-Young(A,i,j-1,x);
            else return false;
    }

    main()
    {
        Search-Young(A,1,n,x);
    }




