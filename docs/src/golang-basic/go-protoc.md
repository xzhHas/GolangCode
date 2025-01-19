---
title: 【Golang】proto生成go的相关文件
shortTitle: 17.如何使用protobuf生成文件
description: 【Golang】proto生成go的相关文件
author: GolangCode
category:
  - Go
tags:
 - Go
date: 2024-05-02
---

## 1、查看proto的版本号

```protobuf
protoc --version
```

## 2、安装protoc-gen-go和protoc-gen-go-grpc

```protobuf
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latests
```

## 3、生成protobuff以及grpc的文件

```protobuf
 protoc --go_out=./ --go-grpc_out=./ *.proto
```

## 4、HelloWorld案例
注：在使用客户端调用服务函数的时候注意不要传nil，不然会发生panic()。

> proto文件，及生成go-grpc

```go
syntax = "proto3";
option go_package = ".;helloworld";

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

message HelloRequest {
  string msg = 1;
}

message HelloReply {
  string msg = 1;
}
```

> client.go  客户端

```go
package main

import (
	"context"
	"fmt"
	"google.golang.org/grpc"
	"log"
	pb "test/yunfuwu/examples/helloworld/helloworld"
)

func main() {
	conn, err := grpc.Dial("0.0.0.0:5001", grpc.WithInsecure())
	if err != nil {
		log.Fatalf("did not connect: %v", err)
	}
	defer conn.Close()
	client := pb.NewGreeterClient(conn)

	msg := "英雄联盟"
	r, err := client.SayHello(context.Background(), &pb.HelloRequest{
		Msg: msg,
	})
	if err != nil {
		//panic(err)
	}
	fmt.Println(r.Msg)
}
```

> server.go   服务端

```go
package main

import (
	"context"
	"fmt"
	"google.golang.org/grpc"
	"log"
	"net"

	pb "test/yunfuwu/examples/helloworld/helloworld"
)

type server struct {
}

func (s *server) SayHello(ctx context.Context, req *pb.HelloRequest) (*pb.HelloReply, error) {
	return &pb.HelloReply{Msg: "hello " + req.Msg}, nil
}
func main() {
	lis, err := net.Listen("tcp", ":5001")
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
		return
	}
	fmt.Println(":5001")
	s := grpc.NewServer()
	pb.RegisterGreeterServer(s, &server{})
	if err := s.Serve(lis); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
}
```

