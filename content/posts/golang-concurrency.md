---
title: "Go 语言并发编程实战"
date: 2026-04-03T14:00:00+08:00
draft: false
description: "深入理解 Go 语言的并发模型，掌握 goroutine 和 channel 的使用技巧"
tags: ["Go", "并发", "编程"]
categories: ["技术"]
showToc: true
tocOpen: true
cover:
  alt: "Go 并发编程"
  relative: false
---

## Go 并发模型简介

Go 语言从设计之初就将并发作为核心特性，其并发模型基于 **CSP（Communicating Sequential Processes）** 理论。

Go 的并发哲学：
> Don't communicate by sharing memory; share memory by communicating.
> 不要通过共享内存来通信，而要通过通信来共享内存。

## Goroutine

Goroutine 是 Go 语言中的轻量级线程，由 Go 运行时管理。

```go
package main

import (
    "fmt"
    "time"
)

func worker(id int) {
    fmt.Printf("Worker %d starting\n", id)
    time.Sleep(time.Second)
    fmt.Printf("Worker %d done\n", id)
}

func main() {
    // 启动 5 个 goroutine
    for i := 1; i <= 5; i++ {
        go worker(i)
    }
    time.Sleep(2 * time.Second)
}
```

## Channel

Channel 是 goroutine 之间通信的管道：

```go
package main

import "fmt"

func sum(s []int, c chan int) {
    total := 0
    for _, v := range s {
        total += v
    }
    c <- total // 发送结果到 channel
}

func main() {
    s := []int{7, 2, 8, -9, 4, 0}
    c := make(chan int)
    
    go sum(s[:len(s)/2], c)
    go sum(s[len(s)/2:], c)
    
    x, y := <-c, <-c // 从 channel 接收
    fmt.Println(x, y, x+y)
}
```

## WaitGroup 使用

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    var wg sync.WaitGroup
    
    for i := 1; i <= 5; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            fmt.Printf("Worker %d completed\n", id)
        }(i)
    }
    
    wg.Wait()
    fmt.Println("All workers done!")
}
```

## Select 语句

`select` 让 goroutine 可以等待多个通信操作：

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    c1 := make(chan string)
    c2 := make(chan string)

    go func() {
        time.Sleep(1 * time.Second)
        c1 <- "one"
    }()
    
    go func() {
        time.Sleep(2 * time.Second)
        c2 <- "two"
    }()

    for i := 0; i < 2; i++ {
        select {
        case msg1 := <-c1:
            fmt.Println("Received", msg1)
        case msg2 := <-c2:
            fmt.Println("Received", msg2)
        }
    }
}
```

## 总结

Go 语言的并发特性非常强大：

| 特性 | 说明 |
|------|------|
| Goroutine | 轻量级协程，创建成本极低 |
| Channel | 类型安全的通信管道 |
| Select | 多路复用，监听多个 channel |
| sync 包 | 提供互斥锁、WaitGroup 等同步原语 |

掌握这些特性，能让你写出高效、简洁的并发程序。
