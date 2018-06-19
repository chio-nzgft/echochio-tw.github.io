---
layout: post
title: google表單的外掛 Form Confirmation Emails
date: 2017-09-14
tags: google doc email
---

測試了一下 , 發現外掛 Form Confirmation Emails 還 OK 紀錄一下

<img src="https://echochio-tw.github.io/images/posts/google-doc/1.png">

<img src="https://echochio-tw.github.io/images/posts/google-doc/2.png">


主旨 : 
```
${"公司名稱"} - ${"聯絡人姓名"} 感謝您的填寫
```

本文內容 :  (最後插入一張圖)

```
<p>Dear ${"聯絡人姓名"} 
<p> 感謝您的填寫 , 您填寫的資訊   :
<p>公司名稱 : ${"公司名稱"}
<p>聯絡人姓名 : ${"聯絡人姓名"}
<p>電子郵件信箱 : ${"電子郵件信箱"}
<p>感興趣產品 : ${"感興趣產品"}
<p>
<p>
<img src="https://www.google.com.tw/images/branding/googlelogo/2x/googlelogo_color_272x92dp.png">
```

管理者及測試者收到的信 : 

<img src="https://echochio-tw.github.io/images/posts/google-doc/3.png">
