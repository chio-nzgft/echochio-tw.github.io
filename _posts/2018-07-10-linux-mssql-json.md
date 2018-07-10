---
layout: post
title: linux 抓 mssql 丟 json
date: 2018-07-10
tags: linux mssql json
---

linux 抓 mssql 丟 json
抓正航的資訊

https://XXX.XXX.XXX.XXX/get_use.php?Info=[正航料號]

回應 :
```
{"Quantity":"2837.0000","SuggestPrice":849,"Long":"62.0000","Width":"35.0000","High":"36.0000"}
```

以下方式自行處理修改

```
<?php
// 主機位址
$host = '192.168.0.X';
// MSSQL PORT
$port = '1433';
// 資料庫名稱
$dbname = 'CHIComp01'; 
// 帳號
$user = 'XXX';
// 密碼
$passwd = 'XXX';
// 取哪個倉庫資訊
$Warehouse = 'A01'; 
// 只是取 hostname
$serverName = $host;

// 取 usl 的 Info 資料
$str = $_GET[Info]
// 連資料庫
$connectionInfo = array( "Database"=>$dbname, "UID"=>$user, "PWD"=>$passwd);
$conn = sqlsrv_connect( $serverName, $connectionInfo );
if( $conn === false ) {
    die( print_r( sqlsrv_errors(), true));
}

// 將輸入資料處理 , 去 enter 及 空白 
$str = str_replace(array("/r/n", "/r", "/n", " "), "", $str);
// 將輸入資料處理 , 只取 35 個字元
$str = substr($str, 0, 35);
// 放入 SQL 語法
$stra = "SELECT [Quantity] FROM [CHIComp02].[dbo].[comWareAmount] WHERE [WareID] like '".$Warehouse."' and [ProdID] like '".$str."'";
// SQL 查詢
$stmt = sqlsrv_query( $conn, $stra );
if( $stmt === false) {
    die( print_r( sqlsrv_errors(), true) );
}

// 將查到的放入 Quan 字串
while( $row = sqlsrv_fetch_array( $stmt, SQLSRV_FETCH_ASSOC) ) {
$Quan = $row['Quantity'];
}

// 放入 SQL 語法
$sql1 = "SELECT [ProdName],[SuggestPrice],[Long],[Width],[High] FROM [CHIComp02].[dbo].[comProduct]";
$sql2 = "Where [CHIComp02].[dbo].[comProduct].[ProdID] like ";
$sql = $sql1.$sql2."'".$str."'";
// SQL 查詢
$stmt = sqlsrv_query( $conn, $sql );
if( $stmt === false) {
    die( print_r( sqlsrv_errors(), true) );
}
// 將查到的放入 data_array 矩陣 ( 包含  Quan 字串 也放入)
while( $row = sqlsrv_fetch_array( $stmt, SQLSRV_FETCH_ASSOC) ) {
$data_array = [
// "ProdName" => $row['ProdName'] ,
"Quantity" =>$Quan,
"SuggestPrice" =>$row['SuggestPrice'],
"Long" =>$row['Long'] ,
"Width" =>$row['Width'],
"High" =>$row['High'],
];
}

sqlsrv_free_stmt( $stmt);
// data_array 矩陣 轉 json 到 data_json 字串
$data_json = json_encode($data_array);
// 列出 data_json
echo $data_json;
?>
```
