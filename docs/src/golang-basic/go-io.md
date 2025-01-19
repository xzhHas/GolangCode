---
title: 关于golang的I/O操作
shortTitle: 16.关于Go的I/O操作
description: 关于golang的I/O操作
author: GolangCode
category:
  - Go
tags:
 - Go
date: 2024-05-01
---

**个人主页：**[席万里的个人空间](https://blog.csdn.net/m0_73337964?spm=1010.2135.3001.5343)

## 1、Fileinfo

查看文件的基本信息Fileinfo，使用Stat方法。

```go
package main

import (
	"fmt"
	"os"
	"path"
	"path/filepath"
)

func main() {
	fileInfo, err := os.Stat("./demo.txt")
	if err != nil {
		fmt.Println("err :", err)
		return
	}
	//文件名
	fmt.Println(fileInfo.Name())
	//文件大小
	fmt.Println(fileInfo.Size())
	//是否是目录
	fmt.Println(fileInfo.IsDir()) //IsDirectory
	//修改时间
	fmt.Println(fileInfo.ModTime())
	//权限
	fmt.Println(fileInfo.Mode()) //-rw-r--r--
}
```

## 2、读写操作(Reader,Writer)

- Reader

`Read函数：func (f *File) Read(b []byte) (n int, err error)`

```go
package main

import (
	"fmt"
	"io"
	"os"
)

func main() {
	fileName := "demo.txt"
	file, err := os.Open(fileName)
	if err != nil {
		fmt.Println("err:", err)
		return
	}
	defer file.Close()

	bs := make([]byte, 4, 4)
	n := -1
	for {
		n, err = file.Read(bs)
		if n == 0 || err == io.EOF {
			fmt.Println("读取到了文件的末尾，结束读取操作。。")
			break
		}
		fmt.Println(string(bs[:n]))
	}
}
```

>  运行结果
>
> ![](https://cdn.golangcode.cn/images/202501181813191.jpeg)


- Writer

`func OpenFile(name string, flag int, perm FileMode) (*File, error)`

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	fileName := "demo.txt"
	file, err := os.OpenFile(fileName, os.O_CREATE|os.O_WRONLY|os.O_APPEND, os.ModePerm)
	if err != nil {
		fmt.Println(err)
		return
	}
	defer file.Close()

	bs := []byte{97, 98, 99, 100} //a,b,c,d
	n, _ := file.Write(bs[:2])
	fmt.Println(n)
	file.WriteString("\n")
	//直接写出字符串
	n, _ = file.WriteString("HelloWorld")
	fmt.Println(n)
	file.WriteString("\n")
	n, _ = file.Write([]byte("today"))
	fmt.Println(n)
}
```

> 运行结果
>
> ![在这里插入图片描述](https://cdn.golangcode.cn/images/202501181814004.jpeg)


## 3、文件复制

在io包中主要是操作流的一些方法，文件复制就是把一个文件复制到另一个目录下。

它的原理就是通过程序，从源文件读取文件中的数据，在写出到目标文件里。

### 1、io包下的Read()和Write()方法实现

```go
func File1(srcFile, destFile string) (int, error) {
	file1, err := os.Open(srcFile)
	if err != nil {
		return 0, err
	}
	file2, err := os.OpenFile(destFile, os.O_WRONLY|os.O_CREATE, os.ModePerm)
	if err != nil {
		return 0, err
	}
	defer file1.Close()
	defer file2.Close()
	//拷贝数据
	bs := make([]byte, 1024, 1024)
	n := -1 //读取的数据量
	total := 0
	for {
		n, err = file1.Read(bs)
		if err == io.EOF || n == 0 {
			fmt.Println("拷贝完毕。。")
			break
		} else if err != nil {
			fmt.Println("报错了。。。")
			return total, err
		}
		total += n
		file2.Write(bs[:n])
	}
	return total, nil
}
```

### 2、io包下的Copy()方法实现

```go
func File2(srcFile, destFile string)(int64,error){
    file1,err:=os.Open(srcFile)
    if err != nil{
        return 0,err
    }
    file2,err:=os.OpenFile(destFile,os.O_WRONLY|os.O_CREATE,os.ModePerm)
    if err !=nil{
        return 0,err
    }
    defer file1.Close()
    defer file2.Close()

    return io.Copy(file2,file1)
}
```

### 3、ioutil包

> 由于使用一次性读取文件，再一次性写入文件的方式，所以该方法不适用于大文件，容易内存溢出。

```go
func File3(srcFile, destFile string)(int,error){
    input, err := ioutil.ReadFile(srcFile)
    if err != nil {
        fmt.Println(err)
        return 0,err
    }

    err = ioutil.WriteFile(destFile, input, 0644)
    if err != nil {
        fmt.Println("操作失败：", destFile)
        fmt.Println(err)
        return 0,err
    }

    return len(input),nil
}
```

## 4、Seeker接口

Seeker是包装基本Seek方法的接口。

```go
type Seeker interface {
        Seek(offset int64, whence int) (int64, error)
}
```

seek(offset,whence),设置指针光标的位置，随机读写文件：

第一个参数：偏移量 第二个参数：如何设置

0：seekStart表示相对于文件开始， 1：seekCurrent表示相对于当前偏移量， 2：seek end表示相对于结束。

```go
const (
    SeekStart   = 0 // seek relative to the origin of the file
    SeekCurrent = 1 // seek relative to the current offset
    SeekEnd     = 2 // seek relative to the end
)
```

示例代码：

```go
package main

import (
	"fmt"
	"io"
	"os"
)

func main() {
	file, _ := os.OpenFile("demo.txt", os.O_RDWR, 0)
	defer file.Close()
	bs := make([]byte, 10)

	file.Read(bs)
	fmt.Println(string(bs))

	file.Seek(4, io.SeekStart)
	file.Read(bs)
	fmt.Println(string(bs))

	file.Seek(2, 0) //也是SeekStart
	file.Read(bs)
	fmt.Println(string(bs))

	file.Seek(3, io.SeekCurrent)
	file.Read(bs)
	fmt.Println(string(bs))

	file.Seek(0, io.SeekEnd)
	file.WriteString("ABC")
}
```

> 运行结果
>
> ![在这里插入图片描述](https://cdn.golangcode.cn/images/202501181814655.jpeg)


## 5、bufio包

bufio 是通过缓冲来提高效率。

io操作本身的效率并不低，低的是频繁的访问本地磁盘的文件。所以bufio就提供了缓冲区(分配一块内存)，读和写都先在缓冲区中，最后再读写文件，来降低访问本地磁盘的次数，从而提高效率。

简单的说就是，把文件读取进缓冲（内存）之后再读取的时候就可以避免文件系统的io 从而提高速度。同理，在进行写操作时，先把文件写入缓冲（内存），然后由缓冲写入文件系统。看完以上解释有人可能会表示困惑了，直接把 内容->文件 和 内容->缓冲->文件相比， 缓冲区好像没有起到作用嘛。其实缓冲区的设计是为了存储多次的写入，最后一口气把缓冲区内容写入文件。

```go
package main

import (
    "os"
    "fmt"
    "bufio"
)

func main() {
    /*
    bufio:高效io读写
        buffer缓存
        io：input/output

    将io包下的Reader，Write对象进行包装，带缓存的包装，提高读写的效率

        ReadBytes()
        ReadString()
        ReadLine()

     */

     fileName:="/Users/ruby/Documents/pro/a/english.txt"
     file,err := os.Open(fileName)
     if err != nil{
        fmt.Println(err)
        return
     }
     defer file.Close()

     //创建Reader对象
     //b1 := bufio.NewReader(file)
     //1.Read()，高效读取
     //p := make([]byte,1024)
     //n1,err := b1.Read(p)
     //fmt.Println(n1)
     //fmt.Println(string(p[:n1]))

     //2.ReadLine()
     //data,flag,err := b1.ReadLine()
     //fmt.Println(flag)
     //fmt.Println(err)
     //fmt.Println(data)
     //fmt.Println(string(data))

     //3.ReadString()
    // s1,err :=b1.ReadString('\n')
    // fmt.Println(err)
    // fmt.Println(s1)
    //
    // s1,err = b1.ReadString('\n')
    // fmt.Println(err)
    // fmt.Println(s1)
    //
    //s1,err = b1.ReadString('\n')
    //fmt.Println(err)
    //fmt.Println(s1)
    //
    //for{
    //  s1,err := b1.ReadString('\n')
    //  if err == io.EOF{
    //      fmt.Println("读取完毕。。")
    //      break
    //  }
    //  fmt.Println(s1)
    //}

    //4.ReadBytes()
    //data,err :=b1.ReadBytes('\n')
    //fmt.Println(err)
    //fmt.Println(string(data))

    //Scanner
    //s2 := ""
    //fmt.Scanln(&s2)
    //fmt.Println(s2)

    b2 := bufio.NewReader(os.Stdin)
    s2, _ := b2.ReadString('\n')
    fmt.Println(s2)

}
```

## 6、ioutil包

```go
package main

import (
    "io/ioutil"
    "fmt"
    "os"
)

func main() {
    /*
    ioutil包：
        ReadFile()
        WriteFile()
        ReadDir()
        ..
     */

    //1.读取文件中的所有的数据
    fileName1 := "/Users/ruby/Documents/pro/a/aa.txt"
    data, err := ioutil.ReadFile(fileName1)
    fmt.Println(err)
    fmt.Println(string(data))

    //2.写出数据
    fileName2:="/Users/ruby/Documents/pro/a/bbb.txt"
    s1:="helloworld面朝大海春暖花开"
    err:=ioutil.WriteFile(fileName2,[]byte(s1),0777)
    fmt.Println(err)

    //3.
    s2:="qwertyuiopsdfghjklzxcvbnm"
    r1:=strings.NewReader(s2)
    data,_:=ioutil.ReadAll(r1)
    fmt.Println(data)

    //4.ReadDir(),读取一个目录下的子内容：子文件和子目录，但是仅有一层
    dirName:="/Users/ruby/Documents/pro/a"
    fileInfos,_:=ioutil.ReadDir(dirName)
    fmt.Println(len(fileInfos))
	rr:=ioutil.WriteFile(fileName2,[]byte(s1),0777)
    fmt.Println(err)
}
```

