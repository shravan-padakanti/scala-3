## Parallelism and collections

**Parallel processing of collections** is important. It is one of the main applications of parallelism today. We examine conditions when this can be done:

* properties of collections: ability to split, combine
* properties of operations: associativity, independence

### Functional programming and collections

Operations on collections are key to functional programming 

* **map**: apply function to each element

    ```scala
    List(1,3,8).map(x => x*x) == List(1, 9, 64)
    ```
* **fold**: combine elements with a given operation

    ```scala
    List(1,3,8).fold(100)((s,x) => s + x) == 112
    ```
* **scan**: combine folds of all list prefixes

    ```scala
    List(1,3,8).scan(100)((s,x) => s + x) == List(100, 101, 104, 112)
    ```

These operations are even more important for parallel than sequential collections: they encapsulate more complex algorithms.

## Choice of data structures

We use `List` to specify the results of operations. **Lists are not good for parallel implementations** because we cannot
efficiently:

* split them in half (need to search for the middle)
* combine them (concatenation needs linear time)

We use these alternatives for now:

* **arrays**: imperative (recall array sum)
* **trees**: can be implemented functionally

Subsequent lectures examine Scala’s parallel collection libraries

* includes many more data structures, implemented efficiently