---
layout: post
title: Execl 抓分隔分類版
date: 2018-06-29
tags: Execl
---

亂寫只為了早點下班

一直複製貼上 , 你改一次我做一次複製貼上 , 那寫程式快多了 .... !!!

```

Dim I1 As Integer, I2 As Integer, L As Integer
'I = 3 to 93
'I = 96 to 143
'I = 146 to 160
'I = 162 to 188
'I = 191 to 198
'A = 1 ,E = 5 ,I = 9,M = 13

Sub all()
Del_Box ("分類版統計")
Sheets.Add After:=Sheets(Sheets.Count)
Sheets(Sheets.Count).Name = "分類版統計"
L = 1

'I = 3 to 93
I1 = 3
I2 = 93
For J = 1 To 13 Step 4
    all_sub (J)
Next J

'I = 96 to 143
I1 = 96
I2 = 143
For J = 1 To 13 Step 4
    all_sub (J)
Next J

'I = 146 to 160
I1 = 146
I2 = 160
For J = 1 To 13 Step 4
    all_sub (J)
Next J

'I = 162 to 188
I1 = 162
I2 = 188
For J = 1 To 13 Step 4
    all_sub (J)
Next J

'I = 191 to 198
I1 = 191
I2 = 198
For J = 1 To 13 Step 4
    all_sub (J)
Next J

Sheets("分類版統計").Select
Cells.Select
Cells.EntireColumn.AutoFit
Cells.EntireRow.AutoFit

lr = Cells(1, 1).End(xlDown).Row
lre = lr
For xlr = lr To 2 Step -1
    If Cells((xlr - 1), 1).Value = Cells(xlr, 1).Value Then
        Cells(xlr, 1).Value = ""
    Else
        Range(Cells(lre, 1), Cells(xlr, 1)).Select
        With Selection
            .HorizontalAlignment = xlGeneral
            .VerticalAlignment = xlCenter
            .WrapText = False
            .Orientation = 0
            .AddIndent = False
            .IndentLevel = 0
            .ShrinkToFit = False
            .ReadingOrder = xlContext
            .MergeCells = True
        End With
        lre = xlr - 1
    End If
Next xlr

        Range(Cells(lre, 1), Cells(1, 1)).Select
        With Selection
            .HorizontalAlignment = xlGeneral
            .VerticalAlignment = xlCenter
            .WrapText = False
            .Orientation = 0
            .AddIndent = False
            .IndentLevel = 0
            .ShrinkToFit = False
            .ReadingOrder = xlContext
            .MergeCells = True
        End With

End Sub

Sub all_sub(F As Integer)
a = ""
B = 0
Sheets("分類版").Select
For I = I1 To I2
    If (Cells(I, F).Font.Color) = 16711680 And Not (Cells(I, F) = "") Then
        a = Cells(I, F).Value
        B = 1
    End If
    If Cells(I, F) = "" Then
        a = ""
        B = 0
    End If
    If Not (Cells(I, F) = "") And Not (Cells(I, F).Font.Color = 16711680) And B = 1 And Not (Cells(I, (F + 3)).Value = "出清/不再引進") Then
        Cells(I, F).Select
        Selection.Copy
        Sheets("分類版統計").Select
        Cells(L, 2).Select
        ActiveSheet.Paste
        Cells(L, 1).Value = a
        L = L + 1
        Sheets("分類版").Select
    End If
Next I
End Sub
Sub Del_Box(Box_data As String)

' 判斷是否是第一個是移到最後一個刪除
 If Box_data = Sheets(1).Name Then
  Sheets(Box_data).Move Before:=Sheets(Sheets.Count)
 End If
' 刪除已存在的暫存
   For I = 2 To Sheets.Count
            If Box_data = Sheets(I).Name Then
              Application.DisplayAlerts = False
              Sheets(I).Delete
              Application.DisplayAlerts = True
              Exit For
             End If
    Next
End Sub
```

