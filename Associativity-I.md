We have already seen the definition of Associativity in the [previous lecture.](https://github.com/rohitvg/scala-parallel-programming-3/wiki/Parallel-fold()#associative-operation).

The consequence was: 

1. Two expressions with same list of operands connected with ⊗, but different parentheses evaluate to the same result ([here](https://github.com/rohitvg/scala-parallel-programming-3/wiki/Parallel-fold()#associative-operation))
2. Reduce on any tree with this list of operands gives the same result ([here](https://github.com/rohitvg/scala-parallel-programming-3/wiki/Parallel-fold()#consequence-stated-as-tree-reduction))

## Commutativity

Operation `f: (A, A) => A` is commutative _iff_ for every `x, y: f(x, y) = f(y, x)`

* There are operations that are associative but not commutative
* There are operations that are commutative but not associative

**For correctness of `reduce`, we need (just) associativity.**

#### Examples of operations that are both associative and commutative

* addition and multiplication of mathematical integers (BigInt) and of exact rational numbers (given as, e.g., pairs of BigInts)
* addition and multiplication modulo a positive integer (e.g. 2^32), including the usual arithmetic on 32-bit Int or 64-bit Long values
* union, intersection, and symmetric difference of sets
* union of bags (multisets) that preserves duplicate elements
* boolean operations &&, ||, exclusive or
* addition and multiplication of polynomials
* addition of vectors
* addition of matrices of fixed dimension

#### using sum: array norm

[Previously](https://github.com/rohitvg/scala-parallel-programming-3/wiki/Running-Computations-in-Parallel#example-computing-p-norm) we have seen the p-norm. So if we want to compute the p-norm of an array, which combination of operations (like `map`, `fold`, `reduce` that we have seen) does sum of powers correspond to?

Answer:  do a `map` followed by a `reduce`.
```scala
redce(map(a, power(abs(_), p)), _ + __
```
Here `+` is the associateive peration of `reduce`. `map` can be combined with `reduce` directly as above to avoid intermediate collections that would be created if we just do a map and thus avoid unnecessary allocations.

#### Examples of operations that are associative but not commutative

* concatenation (append) of lists: (x ++ y) ++ z == x ++ (y ++ z)
* concatenation of Strings (which can be viewed as lists of Char)
* matrix multiplication AB for matrices A and B of compatible dimensions
* composition of relations r ⊙ s = {(a, c) | ∃b.(a, b) ∈ r ∧ (b, c) ∈ s}
* composition of functions (f ◦ g)(x) = f(g(x))

Because they are associative, reduce still gives the same result

#### Examples of operations are commutative but not associative

This function is commutative but not associative:  `f(x, y) = x^2 + y^2`

Indeed in this case, `f(x, y) = x(y, x)`, But:
```
f(f(x, y), z) = (x^2 + y^2)^2 + z^2
f(x, f(y, z)) = x^2 + (y^2 + z^2)^2
```
These are polynomials of different growth rates with respect to different variables and are easily seen to be different for many `x, y, z`.

Proving commutativity alone does not prove associativity and does not guarantee that the result of reduce is the same as e.g. `reduceLeft` and `reduceRight`.

#### Associativity is not preserved by mapping

In general, `if f(x, y)` is commutative and `h1(z)`, `h2(z)` are arbitrary functions, then any function defined by `g(x, y) = h2(f(h1(x), h1(y)))` is equal to `h2(f(h1(y), h2(x))) = g(y, x)`, so it is commutative, but it often loses associativity even if `f` was associative to start with.

Previous example was an instance of this for `h1(x) = h2(x) = x^2`.

**NOTE**: When combining and optimizing `reduce` and `map` invocations, we need to be careful that operations given to reduce remain associative.

#### Floating point addition is commutative but not associative
```scala
scala> val e = 1e-200
e: Double = 1.0E-200
scala> val x = 1e200
x: Double = 1.0E200
scala> val mx = -x
mx: Double = -1.0E200

scala> (x + mx) + e
res2: Double = 1.0E-200
scala> x + (mx + e)
res3: Double = 0.0
scala> (x + mx) + e == x + (mx + e)
res4: Boolean = false
```

#### Floating point multiplication is commutative but not associative
```scala
scala> val e = 1e-200
e: Double = 1.0E-200

scala> val x = 1e200
x: Double = 1.0E200

scala> (e*x)*x
res0: Double = 1.0E200

scala> e*(x*x)
res1: Double = Infinity

scala> (e*x)*x == e*(x*x)
res2: Boolean = false
```

### Making an operation commutative is easy

Suppose we have a binary operation g and a strict total ordering less (e.g. lexicographical ordering of bit representations).

Then this operation is commutative:

```
def f(x: A, y: A) = if (less(y,x)) g(y,x) else g(x,y)
```
Here `f(x,y)==f(y,x)` because:

* if `x==y` then both sides equal `g(x,x)`
* if `less(y,x)` then left sides is `g(y,x)` and it is not `less(x,y)` so right side is also `g(y,x)`
* if `less(x,y)` then it is not `less(y,x)` so left sides is `g(x,y)` and right side is also `g(x,y)`

We know of no such efficient trick for Associativity.