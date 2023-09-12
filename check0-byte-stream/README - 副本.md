# 简介

**check0**是一个关于字节流的实验。这个实验帮助你搭建基本的实验环境，熟悉开发流程和提交流程。

该实验帮助你理解TCP/IP协议栈的**字节流抽象**，并引导你用C++和`Socket`接口设计实现`wget`程序，以及一个可靠的**单机字节流抽象**。  

## 环境搭建

官方说明：[https://stanford.edu/class/cs144/vm_howto/](https://stanford.edu/class/cs144/vm_howto/)

我选择的环境是Windows11 + virtualBox + Linux VM（官方虚拟机模板）

开发环境：宿主机VS Code remote SSH连接本地虚拟机，通过VS Code进行开发，GDB进行调试

虚拟机配置为8G内存 + 4核，该虚拟机的配置会影响之后所开发程序的benchmark结果

### Hint

请不要尝试在虚拟机上安装 Wakatime 插件统计你的编码时长，这会导致你的虚拟机宕机（我曾经确实这么试过）。

请不要尝试同时创建多个虚拟机，官方虚拟机镜像禁止创建多台虚拟机（我猜是因为端口转发和占用的问题，该虚拟机镜像的默认配置是占用宿主机2222端口）。

因为Linux的脆弱性，请在开发完一个模块后使用VCS将你的工作同步到远程VCS服务器上。我选择的是Github。

## Exercise1 Fetch Web Page And Send Email

### Hint

使用`telnet`和服务器简历`HTTP`链接的时候，如果传送的请求不够迅速，服务器会返回超时错误的回报，所以请快点打字。我们注意到`408`是超时标记。

```shell
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>408 Request Timeout</title>
</head><body>
<h1>Request Timeout</h1>
<p>Server timeout waiting for the HTTP request from the client.</p>
</body></html>
```

以下是正确的返回结果

```shell
HTTP/1.1 200 OK
Date: Wed, 30 Aug 2023 03:15:48 GMT
Server: Apache
Last-Modified: Thu, 13 Dec 2018 15:45:29 GMT
ETag: "e-57ce93446cb64"
Accept-Ranges: bytes
Content-Length: 14
Connection: close
Content-Type: text/plain

Hello, CS144!
```

我（目前）不是斯坦福的学生，所以我没有斯坦福的Email账号。我尝试使用`telnet`连接其他邮件服务器。需要注意的是不是所有邮箱服务器都打开了`smtp`服务，这取决于邮箱服务器的配置，比如`UTC`的邮箱服务器只支持`smtps`，而腾讯的`QQ mail`则都支持。

首先确保能`ping`通学校邮箱的发件服务器

```shell
cs144@vm:~/lab0$ ping smtps.utc.fr
PING smtps.utc.fr (195.83.155.8) 56(84) bytes of data.
64 bytes from smtps.utc.fr (195.83.155.8): icmp_seq=1 ttl=46 time=70.8 ms
64 bytes from smtps.utc.fr (195.83.155.8): icmp_seq=2 ttl=46 time=66.8 ms
64 bytes from smtps.utc.fr (195.83.155.8): icmp_seq=3 ttl=46 time=54.4 ms
```

我们学校的邮件服务器只能用`smtps`

```shell
cs144@vm:~/lab0$ telnet 195.83.155.8 smtps
Trying 195.83.155.8...
Connected to 195.83.155.8.
Escape character is '^]'.
```


换成`QQ mail`邮箱试一下

```shell
cs144@vm:~/lab0$ ping smtp.qq.com
PING smtp.qq.com (43.129.255.54) 56(84) bytes of data.
64 bytes from 43.129.255.54 (43.129.255.54): icmp_seq=1 ttl=47 time=294 ms
64 bytes from 43.129.255.54 (43.129.255.54): icmp_seq=2 ttl=47 time=280 ms
64 bytes from 43.129.255.54 (43.129.255.54): icmp_seq=3 ttl=47 time=289 ms
```

发现成功了，`QQ mail`服务器`smtp`和`smtps`都是支持的
```shell
cs144@vm:~/lab0$ telnet 43.129.255.54 smtp
Trying 43.129.255.54...
Connected to 43.129.255.54.
Escape character is '^]'.
220 newxmesmtplogicsvrszb6-0.qq.com XMail Esmtp QQ Mail Server.
^]
telnet> close
Connection closed.
cs144@vm:~/lab0$ telnet 43.129.255.54 smtps
Trying 43.129.255.54...
Connected to 43.129.255.54.
Escape character is '^]'.
^]
telnet> close
Connection closed.
```

## Exercise2 `natcat`

很简单的实验，不多赘述。

## Exercise3 Using `stream socket` to Writing Network Program

该实验的主题是使用`Socket`来在C++中模拟之前在命令行里使用`telnet`访问服务器下载网页的功能。

### Hint

编译的时候，需要进入`build`目录再进行编译。你可以修改测试样例，但请不要擅自修改`CMakeLists.txt`。

如果你感觉测试样例不能全覆盖，你也许会好奇地点开github网站然后查看分支。你发现有很多分支，且其他分支上有一些针对测试样例的提交，而你现在只在main分支上工作。你会怀疑是不是分支选错了。然后你开始合并分支，然后把自己的本地仓库搞得一团糟。。。

是的，我之前确实这么干过，但请相信官方仓库的正确性，请始终在main分支的版本上工作。在后续的实验中，实验文档会一步一步指导你将这些分支陆续合并到main分支上。

这会往之前的实验中补充新的测试样例，你之前的代码可能无法通过新的测试样例，但请有耐心地修正你之前犯过的错误。

注意，提前合并这些分支会打乱你的开发和测试环境。

请求报文的最后一句需要加上两组换行符。
```C++
const string request = "GET " + path + " HTTP/1.1\r\n" + 
    "Host: " + host + "\r\n" + 
    "Connection: close\r\n\r\n";
```

在编写程序之前请先搞清楚`string_view`的概念。错误地使用`string_view`会导致报文乱码，请求头乱码的结果是服务器无法解析请求，返回`400 Bad Request`。

```shell
cs144@vm:~/minnow/build$ ./apps/webget cs144.keithw.org /hello
Function called: get_URL(cs144.keithw.org, /hello)
HTTP/1.1 400 Bad Request
Date: Wed, 30 Aug 2023 07:52:40 GMT
Server: Apache
Content-Length: 226
Connection: close
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>400 Bad Request</title>
</head><body>
<h1>Bad Request</h1>
<p>Your browser sent a request that this server could not understand.<br />
</p>
</body></html>
```

正确的运行结果如下，拿到的是`200 OK`。请不要在`cout`中再额外添加`\n`，这会使测试无法通过。

```shell
cs144@vm:~/minnow/build$ ./apps/webget cs144.keithw.org /hello
Function called: get_URL(cs144.keithw.org, /hello)
HTTP/1.1 200 OK
Date: Wed, 30 Aug 2023 08:40:51 GMT
Server: Apache
Last-Modified: Thu, 13 Dec 2018 15:45:29 GMT
ETag: "e-57ce93446cb64"
Accept-Ranges: bytes
Content-Length: 14
Connection: close
Content-Type: text/plain

Hello, CS144!
```

正确的测试结果

```shell
cs144@vm:~/cs144/minnow$ cmake --build build --target check_webget
Test project /home/cs144/cs144/minnow/build
    Start 1: compile with bug-checkers
1/2 Test #1: compile with bug-checkers ........   Passed   15.42 sec
    Start 2: t_webget
2/2 Test #2: t_webget .........................   Passed    1.02 sec

100% tests passed, 0 tests failed out of 2

Total Test time (real) =  16.45 sec
Built target check_webget
```

## Exercise4 Implement an in-memory reliable byte stream

在读实验需求的时候，你也许会对TCP的这个组件的特性感到疑惑：

在单一设备上实现先进先出字节流队列的意义是什么？此外你还会注意到这个队列不支持多线程并发。他不支持阻塞读和阻塞写，超出capacity的字符将被丢弃。

### Hint

依然是`string_view`，我注意到在2023 Spring中，`Reader::peek() const`这个接口的返回类型从`string`转变为`string_view`，这是一个C++17中引入的新特性。

在开始设计你的`ByteStream`类之前，请务必理解`strig_view`的设计思想。

`string_view`的陷阱是如果你创建了一个针对局部变量`string`的视图（栈空间），当过程调用返回之后，从`string_view`中读取到的字符串信息是未被定义的，如何选择`ByteStream`的底层数据结构至关重要。

在初次实现后，我写的程序benchmark结果如下：

```shell
cs144@vm:~/minnow$ cmake --build build --target check0
Test project /home/cs144/minnow/build
      Start  1: compile with bug-checkers
 1/10 Test  #1: compile with bug-checkers ........   Passed    0.58 sec
      Start  2: t_webget
 2/10 Test  #2: t_webget .........................   Passed    1.02 sec
      Start  3: byte_stream_basics
 3/10 Test  #3: byte_stream_basics ...............   Passed    0.01 sec
      Start  4: byte_stream_capacity
 4/10 Test  #4: byte_stream_capacity .............   Passed    0.02 sec
      Start  5: byte_stream_one_write
 5/10 Test  #5: byte_stream_one_write ............   Passed    0.02 sec
      Start  6: byte_stream_two_writes
 6/10 Test  #6: byte_stream_two_writes ...........   Passed    0.02 sec
      Start  7: byte_stream_many_writes
 7/10 Test  #7: byte_stream_many_writes ..........   Passed    0.07 sec
      Start  8: byte_stream_stress_test
 8/10 Test  #8: byte_stream_stress_test ..........   Passed    0.04 sec
      Start 16: compile with optimization
 9/10 Test #16: compile with optimization ........   Passed    0.19 sec
      Start 17: byte_stream_speed_test
10/10 Test #17: byte_stream_speed_test ...........***Timeout  12.47 sec
```

我尝试将`string`的构造方法进行调整，使用迭代器构造子串。这一调整将程序的执行时间缩短到了之前的十分之一

```c++
cs144@vm:~/minnow$ cmake --build build --target check0
Test project /home/cs144/minnow/build
      Start  1: compile with bug-checkers
 1/10 Test  #1: compile with bug-checkers ........   Passed    3.16 sec
      Start  2: t_webget
 2/10 Test  #2: t_webget .........................   Passed    1.04 sec
      Start  3: byte_stream_basics
 3/10 Test  #3: byte_stream_basics ...............   Passed    0.05 sec
      Start  4: byte_stream_capacity
 4/10 Test  #4: byte_stream_capacity .............   Passed    0.03 sec
      Start  5: byte_stream_one_write
 5/10 Test  #5: byte_stream_one_write ............   Passed    0.02 sec
      Start  6: byte_stream_two_writes
 6/10 Test  #6: byte_stream_two_writes ...........   Passed    0.03 sec
      Start  7: byte_stream_many_writes
 7/10 Test  #7: byte_stream_many_writes ..........   Passed    0.07 sec
      Start  8: byte_stream_stress_test
 8/10 Test  #8: byte_stream_stress_test ..........   Passed    0.03 sec
      Start 16: compile with optimization
 9/10 Test #16: compile with optimization ........   Passed    1.24 sec
      Start 17: byte_stream_speed_test
             ByteStream throughput: 0.05 Gbit/s
10/10 Test #17: byte_stream_speed_test ...........***Failed    1.82 sec
ByteStream with capacity=32768, write_size=1500, read_size=128 reached 0.05 Gbit/s.
Exception: ByteStream did not meet minimum speed of 0.1 Gbit/s.
```

依然不符合要求，我尝试调整底层容器为`string`，benchmark结果又快了将近十倍，顺利通过速度测试

```shell
cs144@vm:~/minnow$ cmake --build build --target check0
Test project /home/cs144/minnow/build
      Start  1: compile with bug-checkers
 1/10 Test  #1: compile with bug-checkers ........   Passed    4.76 sec
      Start  2: t_webget
 2/10 Test  #2: t_webget .........................   Passed    1.05 sec
      Start  3: byte_stream_basics
 3/10 Test  #3: byte_stream_basics ...............   Passed    0.02 sec
      Start  4: byte_stream_capacity
 4/10 Test  #4: byte_stream_capacity .............   Passed    0.02 sec
      Start  5: byte_stream_one_write
 5/10 Test  #5: byte_stream_one_write ............   Passed    0.02 sec
      Start  6: byte_stream_two_writes
 6/10 Test  #6: byte_stream_two_writes ...........   Passed    0.03 sec
      Start  7: byte_stream_many_writes
 7/10 Test  #7: byte_stream_many_writes ..........   Passed    0.06 sec
      Start  8: byte_stream_stress_test
 8/10 Test  #8: byte_stream_stress_test ..........   Passed    0.03 sec
      Start 16: compile with optimization
 9/10 Test #16: compile with optimization ........   Passed    1.11 sec
      Start 17: byte_stream_speed_test
             ByteStream throughput: 0.86 Gbit/s
10/10 Test #17: byte_stream_speed_test ...........   Passed    0.30 sec

100% tests passed, 0 tests failed out of 10

Total Test time (real) =   7.40 sec
Built target check0
```