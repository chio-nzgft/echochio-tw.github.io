---
layout: post
title: golang 遠端清理 印表機 queue 
date: 2018-06-27
tags: golang printer
---

每次都登入到printer server 去 清理 印表機 queue 很麻煩

寫個程式 telnet 去 清 queue ......

最後用 golng 寫 , 這樣 windows, MacOS, linux 都可以執行


```
package main

import (
	"log"
	"os"
	"time"

	"github.com/ziutek/telnet"
)

const timeout = 10 * time.Second

func checkErr(err error) {
	if err != nil {
		log.Fatalln("Error:", err)
	}
}

func expect(t *telnet.Conn, d ...string) {
	checkErr(t.SetReadDeadline(time.Now().Add(timeout)))
	checkErr(t.SkipUntil(d...))
}

func sendln(t *telnet.Conn, s string) {
	checkErr(t.SetWriteDeadline(time.Now().Add(timeout)))
	buf := make([]byte, len(s)+1)
	copy(buf, s)
	buf[len(s)] = '\n'
	_, err := t.Write(buf)
	checkErr(err)
}

func main() {

	typ := "windows"
	dst := "192.168.0.100:23"
	user := "Administrator"
	passwd := "chi"
	
	todo := "del /Q /F /S \"%windir%\\System32\\spool\\PRINTERS\\*.*\""

	t, err := telnet.Dial("tcp", dst)
	checkErr(err)
	t.SetUnixWriteMode(true)

	var data []byte
	switch typ {
	case "windows":
		expect(t, "login: ")
		sendln(t, user)
		expect(t, "password: ")
		sendln(t, passwd)
		expect(t, "Administrator>")
		sendln(t, "net stop spooler")
		expect(t, "Administrator>")
		sendln(t, todo)
		expect(t, "Administrator>")
		sendln(t, "net start spooler")
		data, err = t.ReadBytes('>')
	default:
		log.Fatalln("bad host type: " + typ)
	}
	checkErr(err)
	os.Stdout.Write(data)
	os.Stdout.WriteString("\n")
}
```
