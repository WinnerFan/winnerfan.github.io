---
layout: post
title: Package Problem
tags: Algorithm
---

## 初始化

- 恰好装满背包f\[0\]=0，f\[1...V\]=负无穷
- 未要求装满f\[0...V\]=0

初始化的数组没有任何物品可以放入背包时的合法状态，容量为0的背包可被0价值的恰好装满，其他没有合法的解；

## 01背包

N件V容量背包，0或1件，第i件费用c\[i\]，价值w\[i\]，f\[i\]\[v\]前i件物品放入容量v的背包最大价值
```
f[i][v]=max{f[i-1][v],f[i-1][v-c[i]]+w[i]}
```
子问题前i-1件物品的问题，时空复杂度O(NV)

f\[v\]就是f\[i\]\[v\]，需要获取f\[i-1\]\[v\]、f\[i-1\]\[v-c\[i\]\]
```
for i 1...N
    for v V...0
        f[v]=max{f[v],f[v-c[i]]+w[i]}
```
## 完全背包

N件V容量背包，每件无限
```
f[i][v]=max{f[i-1][v-k*c[i]]+k*w[i]|0<=k*c[i]<=v}

for i 1...N
    for v 0...V
        f[v]=max{f[v],f[v-cost]+weight}
```
第i件最多有V/c\[i\]件，变成V/c\[i\]件价格相同的物品，01背包；上一件或者加选本件
```
f[i][v]=max{f[i-1][v],f[i][v-c[i]]+w[i]}

for i 1...N
    for v 0...V
        f[v]=max{f[v],f[v-c[i]]+w[i]}
```
时间复杂度O(VN)

## 多重背包

N件V容量背包，每件n\[i\]件
```
f[i][v]=max{f[i-1][v-k*c[i]]+k*w[i]|0<=k<=n[i]}
```
拆分成1，2，4...n\[i\]-2^k+1的01背包，时间复杂度O(V\*Σlog n\[i\])