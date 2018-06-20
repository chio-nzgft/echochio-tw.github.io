---
layout: post
title: web-go ...用 Multiple build 建造 docker 將 image 變小
date: 2017-06-12
tags: docker golang
---
golang 的好處不需外部 library

只要編譯好就可以了 ...

例如在 linux 編譯好放入就可執行了 

之前寫的  web-go

web.go : 
```
package main

import (
        "fmt"
        "net/http"
        "github.com/ajstarks/svgo"
        "math/rand"
        "time"
        "log"
)

const arcfmt = "stroke:%s;stroke-opacity:%.2f;stroke-width:%dpx;fill:none"

var colors = []string{"red", "green", "blue", "white", "gray"}

func randarc(canvas *svg.SVG, aw, ah, sw int, f1, f2 bool) {
        begin := rand.Intn(aw)
        arclength := rand.Intn(aw)
        end := begin + arclength
        baseline := ah / 2
        al := arclength / 2
        cl := len(colors)
        canvas.Arc(begin, baseline, al, al, 0, f1, f2, end, baseline,
                fmt.Sprintf(arcfmt, colors[rand.Intn(cl)], rand.Float64(), rand.Intn(sw)))

}

func main() {
        http.Handle("/", http.HandlerFunc(circle))
        err := http.ListenAndServe(":9090", nil)
        if err != nil {
                log.Fatal("ListenAndServe:", err)
        }
}

func circle(w http.ResponseWriter, req *http.Request) {
        w.Header().Set("Content-Type", "image/svg+xml")

        rand.Seed(time.Now().UnixNano())

        width := 800
        height := 800
        aw := width / 2
        maxstroke := height / 10
        literactions := 50
        canvas := svg.New(w)
        canvas.Start(width, height)
        canvas.Rect(0, 0, width, height)
        for i := 0; i < literactions; i++ {
                randarc(canvas, aw, height, maxstroke, false, true)
                randarc(canvas, aw, height, maxstroke, false, false)
        }
        canvas.End()
}
```

用 docker 的  golang:latest 編譯好在放到 docker 內執行
```
Dockerfile :

FROM golang:latest AS build-env
COPY web.go /go/web.go
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go get github.com/ajstarks/svgo
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o web

# final stage
FROM centurylink/ca-certs
COPY --from=build-env /go/web /
ENTRYPOINT ["/web"]
```
這樣 image 就小很多了 .......
