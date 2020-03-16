---
layout: post
---


Machine Vision: Theory, Algorithms, Practicalities (Signal Processing and its Applications)

Davies

Usage of MV is:
* recognition problem (detect and determine characters)
* object location

# 2.2 Image proceessing operations

`P0` is byte image variable in 3x3 window, `P0 = 0..255` and P0 is in center
P4 P3 P2
P5 P0 P1  this is 3x3 window. Left shift is `Q0 = P1` (new image center is P1)
P6 P7 P8
`A0` bit image variable in 3x3 `A0 = 0..1`. Edge detection is sum neighbors
bits and if all are 1 than output is 1.

Sigma values are used for:
Shrink dark objects: only if all neighbors are 1 we leave that pixel, in other
words if next to a light background than expand light background (into dark
object).
```
sigma = A1 + A2 ... + A8
if (sigma < 8) B0 = 0 else B0 = A0
```
Oposite, expand dark object (shring light): if next to dark object than expand
dark object.
```
if (sigma > 0) B0 = 1 else B0 = A0
```
Edge finding for binary images: (if next to all dark, we mark it 0, else
original)
```
if (sigma == 8) B0 = 0 else B0 = A0
```
which can we shown in table (canceling all object except near dark)
       sigma
      0-7  8
A0 0   0   0
   1   1   0
Remove salt noise (single 0 surrounded with 1) and remove pepper noise (remove
binary value 1 surrounded with 0).
```
if (sigma<2) B0 = 0
elsif (sigma > 6) B0 = 1
else B0 = A0
```

# 2.3 Concolutions and Point spread functions
