`fold` combine elements with a given operation:
```scala
List(1,3,8).fold(100)((s,x) => s + x) == 112
```
Variants:
```scala
List(1,3,8).foldLeft(100)((s,x) => s - x) == ((100 - 1) - 3) - 8 == 88 // folds from left
List(1,3,8).foldRight(100)((s,x) => s - x) == 1 - (3 - (8-100)) == -94 // folds from right
List(1,3,8).reduceLeft((s,x) => s - x) == (1 - 3) - 8 == -10
List(1,3,8).reduceRight((s,x) => s - x) == 1 - (3 - 8) == 6
```
To enable parallel operations, we look at associative operations

* addition, string concatenation (but not minus)

## Associative operation

Operation `f: (A,A) => A` is associative _iff_ for every x, y, z: 
```
f(x, f(y, z)) = f(f(x, y), z)
```

If we write f(a, b) in infix form as a ⊗ b, associativity becomes: 
```
x ⊗ (y ⊗ z) = (x ⊗ y) ⊗ z
```

Consequence: consider two expressions with same list of operands connected with ⊗, but different parentheses. Then these expressions evaluate to the same result, for example: 
```
(x ⊗ y) ⊗ (z ⊗ w) = (x ⊗ (y ⊗ z)) ⊗ w = ((x ⊗ y) ⊗ z) ⊗ w
```

### Trees instead of expressions

Each expression built from values connected with our associativity operator ⊗ can be represented as a tree:
* leaves are the values
* nodes are ⊗

![associativity_trees](https://github.com/rohitvg/scala-parallel-programming-3/blob/master/resources/images/associativity_trees.png)

### Folding trees

How do we compute the value of such an expression tree?

``` scala
sealed abstract class Tree[A]
case class Leaf[A](value: A) extends Tree[A]
case class Node[A](left: Tree[A], right: Tree[A]) extends Tree[A]
```
Result of evaluating the expression is given by a reduce of this tree. What is its (sequential) definition?

```scala
def reduce[A](t: Tree[A], f : (A,A) => A): A = t match {
    case Leaf(v) => v
    case Node(l, r) => f(reduce[A](l, f), reduce[A](r, f)) // Node -> function f
}
```
We can think of reduce as replacing the constructor Node with the given function `f`.

**Usage**:
```
    f
   / \
  1   f
     / \
    3   8

```
For non-associative operation, the result depends on structure of the tree:
```scala
def tree = Node(Leaf(1), Node(Leaf(3), Leaf(8)))
def fMinus = (x:Int, y:Int) => x - y
def res = reduce[Int](tree, fMinus) // 6
```

### Parallel reduce of a tree

How to make that tree reduce parallel?

We just use the `parallel` function to reduce left and the right branches parallely.
```scala
def reduce[A](t: Tree[A], f : (A,A) => A): A = t match {
    case Leaf(v) => v
    case Node(l, r) => {
        val (lV, rV) = parallel(reduce[A](l, f), reduce[A](r, f)) // <----
        f(lV, rV)
    }
}
```
**Question**: What is the depth complexity of such reduce?

**Answer**: height of the tree

## Associativity stated as tree reduction

How can we restate associativity of such trees?

![associativity_trees_2](https://github.com/rohitvg/scala-parallel-programming-3/blob/master/resources/images/associativity_trees_2.png)

If f denotes ⊕, in Scala we can write this also as:
```scala
reduce(Node(Leaf(x), Node(Leaf(y), Leaf(z))), f) == reduce(Node(Node(Leaf(x), Leaf(y)), Leaf(z)), f)
```

### Order of elements in a tree

We can convert a tree into a list in order to describe the ordering of elements of a tree:
```scala
def toList[A](t: Tree[A]): List[A] = t match {
    case Leaf(v) => List(v)
    case Node(l, r) => toList[A](l) ++ toList[A](r) 
}
```
Suppose we also have tree map:
```scala
def map[A,B](t: Tree[A], f : A => B): Tree[B] = t match {
    case Leaf(v) => Leaf(f(v))
    case Node(l, r) => Node(map[A,B](l, f), map[A,B](r, f)) 
}
```
Can you express toList using map and reduce?
```scala
toList(t) == reduce(map(t, List(_)), _ ++ _)
```

### Consequence stated as tree reduction

Now that we have an idea about the ordering of the tree, we can see the consequence of the tree reduction: consider two expressions with same list of operands connected with ⊗, but different parentheses. Then these expressions evaluate to the same result. 

Express this consequence in Scala using functions we have defined so far: Consequence (Scala): if function `f : (A, A) => A` is associative, and we have 2 trees `t1:Tree[A]` and `t2:Tree[A]`, mapped to the same list of elements such that `toList(t1) == toList(t2)`, then: `reduce(t1, f)==reduce(t2, f)`

**Explanation**:

Intuition: given a tree, use tree rotation until it becomes list-like.

![associativity_trees_3](https://github.com/rohitvg/scala-parallel-programming-3/blob/master/resources/images/associativity_trees_3.png)

Line 1. `(x ⊕ y) ⊕ z == x ⊕ (y ⊕ z)`
Line 2. `(x ⊕ y) ⊕ (p ⊕ q) == (x ⊕ (y ⊕ (p ⊕ q)))`

Applying rotation to tree preserves `toList` as well as the value of `reduce`.

`toList(t1)==toList(t2)` ===> rotations can bring `t1`, `t2` to be the same tree.

### Towards reduction for arrays

We have seen reduction on trees. Often we work with collections where we only know the ordering and not
the tree structure. How can we do reduction in case of, e.g., arrays, where a tree structure is not immediately visible. 

* convert it into a balanced tree
* do tree reduction

Because of associativity, we can choose any tree that preserves the order of elements of the original collection Tree reduction replaces Node constructor with `f`, so we can just use `f` directly instead of building tree nodes.

When the segment is small, it is faster to process it sequentially.

Here is an implementation of parallel array reduce:

```scala

def reduceSeg[A](inp: Array[A], left: Int, right: Int, f: (A,A) => A): A = {
  if (right - left < threshold) {  // Base case that is handled sequentially
    var res= inp(left); var i= left+1
    while (i < right) { res= f(res, inp(i)); i= i+1 }
    res
  } else {
    val mid = left + (right - left)/2
    val (a1,a2) = parallel(reduceSeg(inp, left, mid, f), reduceSeg(inp, mid, right, f)) // <- parallel recursive
    f(a1,a2)
  }
}
def reduce[A](inp: Array[A], f: (A,A) => A): A = reduceSeg(inp, 0, inp.length, f)
```
