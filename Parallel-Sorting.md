## Merge Sort

### Merge Sort

Merge sort is based on the **divide-and-conquer** paradigm.

1. Divide Step: Divide the given array in two, each containing about half of the elements of the original array.
2. Conquer Step: Conquer by recursively sorting the two subarrays.
3. Combine Step: Combine the elements by merging the two sorted subarrays.

![Merge Sort](https://github.com/rohitvg/scala-parallel-programming-3/blob/master/resources/images/merge_sort.png)

The running time is `O(n)`.

We will implement a **parallel merge sort** algorithm.
1. recursively sort the two halves of the array in parallel
2. sequentially merge the two array halves by copying into a temporary
array
3. copy the temporary array back into the original array
The parMergeSort method takes an array, and a maximum depth: