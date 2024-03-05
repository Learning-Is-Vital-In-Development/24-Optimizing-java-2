### Mark and Sweep

[Mark-and-Sweep: Garbage Collection Algorithm](https://www.geeksforgeeks.org/mark-and-sweep-garbage-collection-algorithm/)

![0_QHc_SXR1fxVDpdrW](https://github.com/rachel5004/24-optimizing-java-2/assets/75432228/ee9b8559-f47e-455c-bc2d-fe07ea96eb15)

#### Mark

> Mark bit : Dead(false, 0) Or Live(true, 1) 를 Marking 해 두기 위한 값. 기본적으로 객체가 생성될때 Mark bit 가 0으로 생성

```C
Mark(root)
If markedBit(root) = false then
                     markedBit(root) = true
                                       For each v referenced by root
                                       Mark(v)
```
