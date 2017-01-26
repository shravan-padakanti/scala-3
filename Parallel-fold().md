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

### Associative operation

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

```scala
def reduce[A](t: Tree[A], f : (A,A) => A): A = t match {
    case Leaf(v) => v
    case Node(l, r) => {
        val (lV, rV) = parallel(reduce[A](l, f), reduce[A](r, f))
        f(lV, rV)
    }
}
```
**Question**: What is the depth complexity of such reduce?

**Answer**: height of the tree