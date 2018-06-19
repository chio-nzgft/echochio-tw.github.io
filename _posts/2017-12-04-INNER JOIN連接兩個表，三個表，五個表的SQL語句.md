---
layout: post
title: INNER JOIN連接兩個表，三個表，五個表的SQL語句
date: 2017-12-04
description: INNER JOIN連接兩個表，三個表，五個表的SQL語句
tag: INNER JOIN
--- 
想說用不到居然用到了 ... 四個表的 ..... XD

連接兩個數據表的用法：

```

FROM Member 
INNER JOIN MemberSort 
ON Member.MemberSort = MemberSort.MemberSort

```
語法格式可以概括為：

```

FROM 表1 
INNER JOIN 表2 
ON表1.字段號=表2.字段號

```

連接三個數據表的用法：

```

FROM（Member 
    INNER JOIN MemberSort 
    ON Member.MemberSort = MemberSort.MemberSort
    ）
INNER JOIN MemberLevel 
ON Member.MemberLevel = MemberLevel.MemberLevel

```

語法格式可以概括為：

```

FROM（表1 
    INNER JOIN 表2 
    ON 表1.字段號=表2.字段號
    ）
INNER JOIN 表3 
ON表1.字段號=表3.字段號

```

連接四個數據表的用法：

```

FROM（
        （Member 
        INNER JOIN MemberSort 
        ON Member.MemberSort = MemberSort.MemberSort
        ）
    INNER JOIN MemberLevel 
    ON Member.MemberLevel = MemberLevel.MemberLevel
    ）
INNER JOIN MemberIdentity 
ON Member.MemberIdentity = MemberIdentity.MemberIdentity

```

語法格式可以概括為：

```

FROM (
        (表1
        INNER JOIN 表2
        ON 表1.字段號=表2.字段號
        ) 
    INNER JOIN 表3 
    ON 表1.字段號=表3.字段號
    ) 
INNER JOIN 表4 
ON Member.字段號=表4.字段號

```

連接五個數據表的用法：

```

FROM (
        (
            (Member 
            INNER JOIN MemberSort 
            ON Member.MemberSort=MemberSort.MemberSort
            ) 
        INNER JOIN MemberLevel 
        ON Member.MemberLevel=MemberLevel.MemberLevel
        ) 
    INNER JOIN MemberIdentity 
    ON Member.MemberIdentity=MemberIdentity.MemberIdentity
    ) 
INNER JOIN Wedlock 
ON Member.Wedlock=Wedlock.Wedlock

```

語法格式可以概括為：

```

FROM (
        (
            (表1 
            INNER JOIN 表2 
            ON 表1.字段號=表2.字段號
            ) 
        INNER JOIN 表3 
        ON 表1.字段號=表3.字段號
        ) 
    INNER JOIN 表4 
    ON Member.字段號=表4.字段號
    ) 
INNER JOIN 表5 
ON Member.字段號=表5.字段號

```

用得的是 :

```

SELECT [CHIComp01].[dbo].[comProduct].[ProdID],
       [CHIComp01].[dbo].[comProduct].[ProdName],
       [CHIComp01].[dbo].[comProductClass].[ClassName],
       [CHIComp01].[dbo].[comWareHouse].[WareHouseName],
       [CHIComp01].[dbo].[comWareAmount].[Quantity]
          
FROM (

([CHIComp01].[dbo].[comProduct]
INNER JOIN [CHIComp01].[dbo].[comProductClass]
ON [CHIComp01].[dbo].[comProduct].[ClassID]=[CHIComp01].[dbo].[comProductClass].[ClassID])
  
INNER JOIN [CHIComp01].[dbo].[comWareAmount]
ON [CHIComp01].[dbo].[comProduct].[ProdID]=[CHIComp01].[dbo].[comWareAmount].[ProdID])

INNER JOIN [CHIComp01].[dbo].[comWareHouse] 
ON [CHIComp01].[dbo].[comWareHouse].[WareHouseID]=[CHIComp01].[dbo].[comWareAmount].[WareID]

```

