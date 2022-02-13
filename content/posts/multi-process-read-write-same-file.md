+++ 
draft = true
date = 2021-11-13T21:20:16+08:00
title = "多个进程同时操作同一文件"
description = ""
slug = "" 
tags = []
categories = ["Linux"]
externalLink = ""
series = []
+++

最近遇到一个问题，`Server` 所在容器和 `Engine` 容器通过一个 `progress.json` 文件进行交互，会偶现 `progress.json` 文件变空，或有时 `Server` 内读到了错误的 progress. 原因在于两个进程使用了文件这种不可靠的进程间通信方式。所以相应的解决方案就是加文件锁（如 `fcntl`），或使用可靠的 IPC 方式（如 socket）。

首先我们来看下多个进程同时操作同一文件的场景的问题。

# 同时读
多个进程同时读取同一个文件不会出现问题。

# 一个读一个写

`testwrite.go`

```go
func writeFile(filename string, data string) {
	f, _ := os.OpenFile(filename, os.O_CREATE|os.O_WRONLY, 0644)
	defer f.Close()
	body := []byte(data)
	_, _ = f.Write(body)
}

func main() {
	// 首先向文件中写入 “Hello World!”
	writeFile("test.txt", "Hello World!")
	time.Sleep(7 * time.Second)
	// 七秒后，修改文件内容，写入 “Author MeiK!”
	writeFile("test.txt", "Author MeiK!")
}
```

`testread.go`

```go
func readFile(filename string) {
	f, _ := os.OpenFile(filename, unix.O_RDONLY, 0644)
	defer f.Close()

	body := make([]byte, 1)
	n := 1
	for n != 0 {
		time.Sleep(time.Second)
		var err error
		n, err = f.Read(body)
		if err == io.EOF {
			break
		}
		s, _ := f.Seek(0, os.SEEK_CUR)
		fmt.Printf("%c %d\n", body, s)
	}
}

func main() {
	readFile("test.txt")
}
```

同时执行两个程序

```shell
./testwrite & ./testread
```

输出：

```
[H] 1
[e] 2
[l] 3
[l] 4
[o] 5
[ ] 6
[ ] 7
[M] 8
[e] 9
[i] 10
[K] 11
[!] 12
```

这个程序打印了读取到的内容以及读取到每一步的文件偏移量。我们首先写入 Hello World!，开始每秒读取一个字符，并且在 7 秒后重新将 Author MeiK! 写入文件。我们最终读取到了什么呢？既不是 Hello World!，也不是 Author MeiK!，而是 Hello MeiK!。我们每个字符串读取到了一半！

从每一步的文件偏移量来看，读取的程序只是按部就班的一个字符一个字符的读取文件，对文件内容的变化毫无感知，当读取到文件结尾的 EOF 时结束读取。

那么我们要如何保证读取与写入的一致性呢？ Linux 提供了 fcntl 系统调用，可以锁定文件。

我们对刚刚的文件稍作修改，使用 fcntl 进行加锁：



# 同时写

# 参考
- Linux 中多个进程操作同一个文件时会发生什么, https://meik2333.com/posts/linux-many-proc-write-file/

