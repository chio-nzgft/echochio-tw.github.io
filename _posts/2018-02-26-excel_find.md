---
layout: post
title: excel 找相同資料複製哪行
date: 2018-02-26
tags:  Excel
---

```
Sub aa()
' 從第五行開始找 
' 比對第二個欄位
'找 1 Cells(5,2)-Cells(X,2) 值
'在 2 Cells(5,2)-Cells(X,2) 值
 Worksheets("1").Activate
   lr = Cells(5, 2).End(xlDown).Row
 Worksheets("2").Activate
  lrd = Cells(5, 2).End(xlDown).Row


'雙迴圈找相同 放入那一行
  For xlrd = 5 To lrd
   Worksheets("2").Activate
    strData = Cells(xlrd, 2).Value
 
    For xlr = 5 To lr
      Worksheets("1").Activate
      If strData = Cells(xlr, 2).Value Then
      DDR = xlr & ":" & xlr
      Rows(xlr).Select
      Application.CutCopyMode = False
     Selection.Copy
     Sheets("2").Select
     DDRS = xlrd & ":" & xlrd
     Rows(DDRS).Select
     ActiveSheet.Paste
     exit for
      End If
    Next xlr
 Next xlrd
End Sub

```
亂寫就對了啦 .....XD
