---
title: 网络编程
shortTitle: 5.网络编程
date: 2024-08-27
category:
  - 计算机网络
tag:
  - 计算机网络
---

## 1、Socket编程对应TCP三次握手连接

<img src="https://golang-code.oss-cn-beijing.aliyuncs.com/images/202501071555549.png" alt="image-20241109171133937" style="zoom:67%;" />

Socket编程中connect、bin、listen、accept函数的作用。

服务端首先会通过bind函数给Socket绑定端口和IP地址，然后调用listen函数，进行监听，然后调用accept函数等待客户端建立连接。

当客户端调用connect函数的时候，内核会随机生成初始化序列号，放到TCP报文头部的序号字段中，同时把SYN标志设置为1，这样就表示SYN报文。接着把这个SYN报文发送给服务端，之后客户端处于SYN_SENT状态。

服务端收到SYN报文后，首先服务端也会随机生成初始化序列号，放到报文头部的序号字段中，然后对客户端的初始化序号+1作为确认号，放到TCP报文头部确认应答字段中，并将SYN和ACK标志设置为1，这样就表示SYN-ACk报文，后把该报文发给客户端，之后服务端处于SYN_RCVD状态。

客户端收到服务端SYN-ACK报文后，客户端会回一个ACK确认报文，该报文的确认号是服务端的初始化序号+1，并且ACK标志会设置为1，之后客户端处于ESTABLISHED状态，这时候connect函数就返回了。

服务端收到ACK确认报文后，服务端也进入处于ESTABLISHED状态，这时候accept函数就返回已建立连接的Socket了，后续与客户端进行通信，就是对这个Socket进行读写操作了。

## 2、建立TCP连接，Socket在TCP握手那个阶段可以拿到连接？

<img src="https://golang-code.oss-cn-beijing.aliyuncs.com/images/202501071555058.png" alt="image-20241109172841231" style="zoom:50%;" />

需要完成三次握手后，服务端才能通过accept函数得到已建立TCP连接的Socket。

## 3、listen的参数backlog意义是？

linux内核中会维护两个队列：

- 半连接队列：接收到一个SYN建立连接请求，处于SYN_RCVD状态。（SYN队列）
- 全连接队列：已完成TCP三次握手过程，处于ESTABLISHED状态。（Accept队列）
