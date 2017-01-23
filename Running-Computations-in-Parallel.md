## Basic parallel construct
Given expressions e1 and e2, how do we compute them in parallel and return the pair of results.

```
parallel(e1, e2)

         e1
     / ------ \      
o---            --->
     \ ------ /
         e2
```

To understand this, lets take a look at p-norm.

### Example: Computing p-norm
Given a vector as an array (of integers), compute its p-norm

A p-norm is a generalization of the notion of length from geometry 2-norm of a two-dimensional vector (a1, a2) is `(a1^2 + a2^2)^(1/2)`.

So we have 2 operations here. The sum of squares of the coordinates is operation1 and square root of the result is operation2.

#### n-dimensional vector
Let us represent the n-dimensional vector by an array. We can do the above computaion in a "parallel" by going from `0 to m-1` and `m to n-1` separately and performing operation1 on both simultanieously, and then joining the results and doing operation2 on it. 

Thus, here we split the array into 2 segments and in parallel compute the operation1, and the join the results and compute the operation2: `parallel(sum1, sum2)`

How do we do this if we want to break the array in 4 segments? `parallel(parallel(sum1, sum2), parallel(sum3, sum4))`

We can **generalize this using recursion**.

