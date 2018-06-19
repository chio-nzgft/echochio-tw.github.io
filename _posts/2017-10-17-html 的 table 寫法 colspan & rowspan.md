---
layout: post
title: html 的 table 寫法 colspan & rowspan
date: 2017-10-17
tags: html table colspan rowspan
---
紀錄一下 html 的 table 寫法 .... 

----------
```
<table border="1" cellpadding="5" style="border:2px #00DBDB solid;text-align:center;">
<tr>
<td>表格欄位 1 </td>
<td>表格欄位 2 </td>
</tr>
<tr>
 <td colspan=2> 使用 colspan 欄位</td>
</tr>

<tr>
 <td>表格欄位 3 </td>
 <td rowspan="2">使用 rowspan 的欄位</td>
</tr>
<tr>
 <td>表格欄位 4 </td>
</tr>
</table>
```
----------

<img src="https://echochio-tw.github.io/images/posts/html_table/1.png">
