`fold` combine elements with a given operation:
```scala
List(1,3,8).fold(100)((s,x) => s + x) == 112
```
Variants:
```scala
List(1,3,8).foldLeft(100)((s,x) => s - x) == ((100 - 1) - 3) - 8 == 88
List(1,3,8).foldRight(100)((s,x) => s - x) == 1 - (3 - (8-100)) == -94
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

