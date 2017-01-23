### A Method to Estimate π (3.14…)
Consider a square of side length = 2 and a circle of diameter = 2. The circle is inside the square.

Ratio between the surfaces of 1/4 of a circle and 1/4 of a square:
```
λ = ( pi*(1^2)/2 / (2^2)/4 )
λ = pi/4
```

Estimating λ: randomly sample points inside the square.
Count how many fall inside the circle. Multiply this ratio by 4 for an estimate of `pi`

```scala
import scala.util.Random

def mcCount(iter: Int): Int = {
    val randomX = new Random
    val randomY = new Random
    var hits = 0
    for (i <- 0 until iter) {
        // since we are in the quarter of the cicle/square we get coordinates from 0 to 1
        val x = randomX.nextDouble // in [0,1] 
        val y = randomY.nextDouble // in [0,1]
        if (x*x + y*y < 1) hits= hits + 1
    }
    hits
}
```
### Sequential Code for Sampling Pi

```scala
def monteCarloPiSeq(iter: Int): Double = 4.0 * mcCount(iter) / iter
```

### Four-Way Parallel Code for Sampling Pi
```scala
def monteCarloPiPar(iter: Int): Double = {
    val ((pi1, pi2), (pi3, pi4)) = parallel( parallel(mcCount(iter/4), mcCount(iter/4)),
                                             parallel(mcCount(iter/4), mcCount(iter - 3*(iter/4))) )
    4.0 * (pi1 + pi2 + pi3 + pi4) / iter
}