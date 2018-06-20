---
layout: post
title: python 的 lambda, reduce, filter
date: 2017-06-22
tags: python lambda reduce filter
---

先看範例 
看到很簡單 ..用 map 放參數 呼叫 函數
```
>>> double_func = lambda s : s * 2
>>> map(double_func, [1,2,3,4,5])
[2, 4, 6, 8, 10]
```

用 for loop 就比較明白了 .....
```
>>> for x in range(1, 6): double_func(x)
...
2
4
6
8
10
```

reduce 就是把結果再放進去當參數
```
>>> reduce(plus, [1,2,3,5,5])
16
```

就是 "plus"
```
>>> 1+2+3+5+5
16
```

 reduce + map 就是放參數 ....當然可放兩個
```
>>> plus = lambda x,y : (x or 0) + (y or 0)
>>> map(plus, [1,2,3,4], [4,5,6])
[5, 7, 9, 4]
```

filter 就是把 true 列出來....... 就是 "1"
```
>>> mode2 = lambda x : x % 2
>>> filter(mode2, [1,2,3,4,5,6,7,8,9,10])
[1, 3, 5, 7, 9]
>>> map(mode2, [1,2,3,4,5,6,7,8,9,10])
[1, 0, 1, 0, 1, 0, 1, 0, 1, 0]
```

函數變化一下 ......裡面用 and , or 回不同結果

```
>>> pr = lambda s:s
>>> print_num = lambda x: (x==1 and pr("one")) or (x==2 and pr("two")) or (pr("other"))
>>> print_num(1)
'one'
>>> print_num(2)
'two'
>>> print_num(3)
'other'
```

說了一大堆好像很簡單 ....混在一起 .....哈看不懂 !!!!
(網路有名範例 兩數找笛卡兒積 .... 找出大於 25 的)

```
>>> bigmuls = lambda xs,ys: filter(lambda (x,y):x*y > 25, combine(xs,ys))
>>> combine = lambda xs,ys: map(None, xs*len(ys), dupelms(ys,len(xs)))
>>> dupelms = lambda lst,n: reduce(lambda s,t:s+t, map(lambda l,n=n: [l]*n, lst))
>>> print bigmuls([1,2,3,4],[10,15,3,22])
[(3, 10), (4, 10), (2, 15), (3, 15), (4, 15), (2, 22), (3, 22), (4, 22)]
```
