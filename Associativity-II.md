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

As we know, commutativity of `f` alone does not imply associativity. But it does implu it if we have an additional property. Define:
```
E(x,y,z) = f(f(x,y), z)
```
We say arguments of E can rotate if `E(x,y,z) = E(y,z,x)`, that is:
```
f(f(x,y), z) = f(f(y,z), x)
```
**Claim**: if f is commutative and arguments of E can rotate then f is also associative.

**Proof**: `f(f(x,y), z) = f(f(y,z), x) = f(x, f(y,z))`


