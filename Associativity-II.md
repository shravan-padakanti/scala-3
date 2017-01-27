## Associative operations on tuples

Suppose `f1: (A1,A1) => A1` and `f2: (A2,A2) => A2` are associative

Then `f: ((A1,A2), (A1,A2)) => (A1,A2)` defined by `f((x1,x2), (y1,y2)) = (f1(x1,y1), f2(x2,y2))` is also associative:
```
f(f((x1,x2), (y1,y2)), (z1,z2)) ==
f((f1(x1,y1), f2(x2,y2)), (z1,z2)) ==
(f1(f1(x1,y1), z1), f2(f2(x2,y2), z2)) == (because f1, f2 are associative)
(f1(x1, f1(y1,z1)), f2(x2, f2(y2,z2))) ==
f((x1 x2), (f1(y1,z1), f2(y2,z2))) ==
f((x1 x2), f((y1,y2), (z1, z2)))
```
We can similarly construct associative operations on for n-tuples.

### Examples

#### 1. Rational Multiplication

Suppose we use 32-bit numbers to represent numerator and denominator of a rational number. Then we can define multiplication working on pairs of numerator and denominator
```
times((x1,y1), (x2, y2)) = (x1*x2, y1*y2)
```
Because multiplication modulo `2^32` is associative and hence so is `times`.

#### 2. Average

Given a collection of integers, compute the average
```scala
val sum = reduce(collection, _ + _)
val length = reduce(map(collection, (x:Int) => 1), _ + _)
sum/length
```
This includes two reductions. Is there a solution using a single reduce?

**Solution**: Use pairs that compute sum and length at once
```scala
f((sum1,len1), (sum2, len2)) = (sum1 + sum1, len1 + len2)
```
Function f is associative because addition is associative.
Solution is then:
```scala
val (sum, length) = reduce(map(collection, (x:Int) => (x,1)), f)
sum/length
```

## Associativity through symmetry and commutativity

As we know, commutativity of `f` alone does not imply associativity. But we can imply it if we have an additional property. Define:
```
E(x,y,z) = f(f(x,y), z)
```
We say arguments of E can rotate if `E(x,y,z) = E(y,z,x)` ( rotates anti clockwise ), that is: 
```
f(f(x,y), z) = f(f(y,z), x)
```
**Claim**: if `f` is commutative **and** arguments of E can rotate then f is also associative.

**Proof**: `f(f(x,y), z) = f(f(y,z), x) = f(x, f(y,z))`

#### Example: addition of modular fractions
```
plus((x1,y1), (x2, y2)) = (x1*y2 + x2*y1, y1*y2)
```
where * and + are all modulo some base (e.g. 2^32).
We can have overflows in both numerator and denominator. Is such plus associative?

Observe that plus is commutative. 

Moreover:
```
E((x1,y1), (x2,y2), (x3,y3)) == 
plus(plus((x1,y1), (x2,y2)), (x3,y3)) ==
plus((x1*y2 + x2*y1, y1*y2), (x3,y3)) == 
((x1*y2 + x2*y1)*y3 + x3*y1*y2, y1*y2*y3) ==
(x1*y2*y3 + x2*y1*y3 + x3*y1*y2, y1*y2*y3)
```
Therefore
```
E((x2,y2), (x3,y3), (x1,y1)) == (x2*y3*y1 + x3*y2*y1 + x1*y2*y3, y2*y3*y1)
```
which is the same. By previous claim, plus is associative.

#### Example: relativistic velocity addition

Let `u, v` range over rational numbers in the open interval `(−1, 1)`. Define `f` to add velocities according to special relativity:
```
f(u, v) = (u + v)/(1 + uv)
```
Clearly, f is commutative: f(u, v) = f(v, u).

f(f(u, v),w) = ((u+v)/(1+uv) + w)/(1 + (u+v)w/(1+uv))
             = (u + v + w + uvw)/(1 + uv + uw + vw)
```
We can rotate arguments `u,v,w`.

If we implement the `f(u, v) = (u + v)/(1 + uv)` using g floating point numbers, then the operation is not associative.

Even though the difference between `f(x, f(y, z))` and `f(f(x, y), z)` is small in one step, over many steps it accumulates, so the result of the reduceLeft and a reduce may differ substantially.

## A family of associative operations on sets

A function including intersection of sets is show to be associative.





