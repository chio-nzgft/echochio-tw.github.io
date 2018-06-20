---
layout: post
title: golang 讀出資料庫到 2D Srting
date: 2017-07-05
tags: golang Srting
---
程式在這裡 :

https://raw.githubusercontent.com/chio-nzgft/OCS-db-to-Gsheet/master/OCS-db-print.go 

不知有沒有更好了寫法 

```
package main

import (
    "database/sql"
    _ "github.com/Go-SQL-Driver/MySQL"
    "fmt"
    "os"
)
//set db info
const (
    DB_HOST = "tcp(127.0.0.1:3306)"
    DB_NAME = "ocsdb"
    DB_USER = "ocsuser"
    DB_PASS = "ocspass"
)

// DB to 2D string
func DBtoString(rows *sql.Rows,len_cols int, len_count int)([][]string) {
    rawResult := make([][]byte, len_cols)
    result := make([][]string,  len_cols)
     for i := range result {
        result[i] = make([]string, len_count)
    }

    dest := make([]interface{}, len_cols)
    for i, _ := range rawResult {
        dest[i] = &rawResult[i]
    }
    var k int
    k = -1
    for rows.Next() {
        k = k +1
        err := rows.Scan(dest...)
        checkErr("Failed to scan row", err)

        for i, raw := range rawResult {
            if raw == nil {
                result[i][k] = "\\N"
            } else {
                result[i][k] = string(raw)
            }
        }
    }
 return result
}

//Get select count line
func checkCount(rows *sql.Rows) (count int) {
        for rows.Next() {
        err:= rows.Scan(&count)
        checkErr("Fail to get count ",err )
    }
    return count
}

//Get select table rows
func checkRow(rows *sql.Rows)(int) {
        cols, err:=  rows.Columns()
        checkErr("Failed to get columns",err )
        return len(cols)
}

//Show error & exit
func checkErr(w_msg string, err error) {
    if err != nil {
       fmt.Println(w_msg,err)
     os.Exit(2)
     return
    }
}

func main() {
    var len_count,len_cols int
    
    // connect db
    dsn := DB_USER + ":" + DB_PASS + "@" + DB_HOST + "/" + DB_NAME + "?charset=utf8"
    db, err := sql.Open("mysql", dsn)
    checkErr("Failed to connect db",err)
    defer db.Close()

    //query get count 
    rows_count, err := db.Query("select COUNT(*) as count from accountinfo")
    checkErr("Failed to run query for count", err)
    len_count = checkCount(rows_count)
    defer rows_count.Close()

    //query get rows
    rows, err := db.Query("select * from accountinfo")
    checkErr("Failed to run query", err)
    len_cols = checkRow(rows)
    defer rows.Close()

    //set up 2D string by len_cols & len_count
    result := make([][]string,  len_cols)
     for i := range result {
        result[i] = make([]string, len_count)
     }

    //Use DB to get 2D String
    result =  DBtoString(rows,len_cols,len_count)

    //juest print 2D String 
    for i:=0 ; i < len_count ; i++{
      for j:=0 ; j < len_cols ; j++{
       fmt.Print(result[j][i])
       fmt.Print(" ")
      }
      fmt.Println("")
   }
}
```
