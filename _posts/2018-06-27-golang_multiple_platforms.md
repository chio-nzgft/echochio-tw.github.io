---
layout: post
title: golang 編譯各平台
date: 2018-06-27
tags: golang
---

GOOS：目標平臺的作業系統（darwin、freebsd、linux、windows） 

GOARCH：目標平臺的體系架構（386、amd64、arm） 

交叉編譯不支持 CGO 所以要禁用它

可編譯 64 位元可執行程式

你當然應該也會使用 386 編譯 32 位元可執行程式 

Go 新版本已支援所有平臺


golang 在 Windows 下編譯 MacOS 64 位元可執行程式

```
SET CGO_ENABLED=0
SET GOOS=darwin
SET GOARCH=amd64
go build main.go
```

golang 在 Windows 下編譯 Linux 64 位元可執行程式

```
SET CGO_ENABLED=0
SET GOOS=linux
SET GOARCH=amd64
go build main.go
```

Linux 下編譯 Windows 32位元系統下可執行程式
```
$ CGO_ENABLED=0 GOOS=windows GOARCH=386 go build test.go
```

Linux 下編譯 Windows 64位元系統下可執行程式
```
$ CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build test.go
```

Linux 下編譯基本是 :

```
env GOOS=target-OS GOARCH=target-architecture go build package-import-path
```

Raspberry Pi的ARM CPU

```
export GOARCH=arm
export GOOS=linux
export GOARM=7
```

請參考下表 :



|GOOS - 目的作業系統 | GOARCH - 目的平台 |
| ------------------------------ | ------------------------ |
android | arm |
darwin | 386 |
darwin | amd64 |
darwin | arm |
darwin | arm64 |
dragonfly | amd64 |
freebsd | 386 |
freebsd | amd64 |
freebsd | arm |
linux | 386 |
linux | amd64 |
linux | arm |
linux | arm64 |
linux | ppc64 |
linux | ppc64le |
linux | mips |
linux | mipsle |
linux | mips64 |
linux | mips64le |
netbsd | 386 |
netbsd | amd64 |
netbsd | arm |
openbsd | 386 |
openbsd | amd64 |
openbsd | arm |
plan9 | 386 |
plan9 | amd64 |
solaris | amd64 |
windows | 386 |
windows | amd64 |
