---
layout: post
title: opencart 加發票測試
date: 2018-07-25
tags: opencart
---

Opencart的路由流程以及MVC如何在opencart中工作。
====================================================

 MVC不是一個應用程序，基本上它遵循設計模式並基於分層架構。

MVC (Model View Controller)

Opencart 模型 視圖 控制器

是三層的，用於以明確定義的方式將數據相互傳遞。

<img src="/images/posts/opencart/1.png">

Controller：在Opencart 的Controller扮演調解者的角色，管理整個程序控制。

當任何客戶通過瀏覽器點擊URL時，將調用控制器文件。

在控制器內部，我們可以加載模型並調用該模型文件的方法來獲取相關數據。從模型文件控制器獲得響應後，將數據發送到視圖文件。

我們還可以在Opencart的控制器中包含JS和CSS文件。

```
admin/coneroller/{..directory_name..}/{..file_name..}.php
catalog/coneroller/{..directory_name..}/{..file_name..}.php
```

Model：用於通過使用SQL查詢從數據庫獲取數據並將相同數據返回給控制器的模型文件。

模型文件主要用於通過SQL命令執行某些操作，如DDL（數據定義語言，即創建，修改，刪除，截斷等）和DML（數據操作語言，即選擇，插入，更新，刪除，合併等）。 

```
admin/model/{..directory_name..}/{..file_name..}.php
catalog/model/{..directory_name..}/{..file_name..}.php
```

View：查看文件以php，json等格式從控制器文件接收結果數據，並將該數據查看到瀏覽器。

您可以在opencart中的視圖文件中編寫HTML，JS / JQuery，Css和PHP代碼。

我們對視圖文件使用.tpl擴展。

```
admin/view/template/{..directory_name..}/{..file_name..}.tpl
catalog/view/template/{..template_name,theme_name..}/{..directory_name..}/{..file_name..}.tpl
```

Language: 我們還使用一個名為language file的文件來進行與數組索引的sting映射。 

Controller通過使用字符串轉換獲取所有索引並將這些數組索引呈現給.tpl文件來加載此語言文件。

```
admin/language/{..lanaguage_name..}/{..directory_name..}/{..file_name..}.php
catalog/language/{..lanaguage_name..}/{..directory_name..}/{..file_name..}.php
```


## Opencart的初始化

startup.php文件中，有一個名為modify（$ filename）的方法，只有在使用ocmod.xml文件時才會返回修改文件的完整匹配路徑：

modification 這 function

```
// Modification Override
function modification($filename) {
        if (defined('DIR_CATALOG')) {
                $file = DIR_MODIFICATION . 'admin/' .  substr($filename, strlen(DIR_APPLICATION));
        } elseif (defined('DIR_OPENCART')) {
                $file = DIR_MODIFICATION . 'install/' .  substr($filename, strlen(DIR_APPLICATION));
        } else {
                $file = DIR_MODIFICATION . 'catalog/' . substr($filename, strlen(DIR_APPLICATION));
        }

        if (substr($filename, 0, strlen(DIR_SYSTEM)) == DIR_SYSTEM) {
                $file = DIR_MODIFICATION . 'system/' . substr($filename, strlen(DIR_SYSTEM));
        }

        if (is_file($file)) {
                return $file;
        }

        return $filename;
}
```

在system / startup.php文件中找到start（'catalog'），該文件將加載framework.php文件，其中創建所有使用的類的對象並將其設置為註冊表。

可以通過它訪問控制器中的所有系統類。

##Controller Architecture 控制器架構

您將找到如下的類名：


