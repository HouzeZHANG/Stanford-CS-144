# Check6 Notes

## Introduction

这个实验的目的是组装并验证**TCP/IP协议栈**

为了排除潜在的因为**NAT**带来的地址转换问题，实验提供了专门的**relay server**来连接两个TCP主机

这个实验很简单，只需要在命令行中操作即可

基于前面所实现的六大组件：

- 传输层的reassember，bytestream，TCP-sender，TCP-receiver

- IP层的routing

- 以及链路层和传输层之间的IP/Ethernet Interface

## Hint

### 端口号需要加一

请注意client和server需要填写**不同的**relay server端口才能正常运行（X和X+1）

```commandline
cs144@vm:~/minnow/build/apps$ ./endtoend client cs144.keithw.org 3001
DEBUG: Network interface has Ethernet address 02:00:00:c8:c7:8b and IP address 192.168.0.1
DEBUG: Network interface has Ethernet address 02:00:00:4f:a6:e5 and IP address 10.0.0.192
DEBUG: adding route 192.168.0.0/16 => (direct) on interface 0
DEBUG: adding route 10.0.0.0/8 => (direct) on interface 1
DEBUG: adding route 172.16.0.0/12 => 10.0.0.172 on interface 1
DEBUG: Network interface has Ethernet address 92:09:4d:93:d2:31 and IP address 192.168.0.50
DEBUG: Connecting from 192.168.0.50:45795...
DEBUG: Connecting to 172.16.0.100:1234...
Successfully connected to 172.16.0.100:1234.
```

简单传送一些字节试试

```commandline
cs144@vm:~/minnow/build$ ./apps/endtoend server cs144.keithw.org 3000
DEBUG: Network interface has Ethernet address 02:00:00:b0:99:a6 and IP address 172.16.0.1
DEBUG: Network interface has Ethernet address 02:00:00:2a:af:4b and IP address 10.0.0.172
DEBUG: adding route 172.16.0.0/12 => (direct) on interface 0
DEBUG: adding route 10.0.0.0/8 => (direct) on interface 1
DEBUG: adding route 192.168.0.0/16 => 10.0.0.192 on interface 1
DEBUG: Network interface has Ethernet address da:c1:1f:0b:7d:ac and IP address 172.16.0.100
DEBUG: Listening for incoming connection...
New connection from 192.168.0.50:34883.
hello i am the server

hello i am the client

cs144@vm:~/minnow/build$ ./apps/endtoend client cs144.keithw.org 3001
DEBUG: Network interface has Ethernet address 02:00:00:c7:5e:1b and IP address 192.168.0.1
DEBUG: Network interface has Ethernet address 02:00:00:8f:7e:d1 and IP address 10.0.0.192
DEBUG: adding route 192.168.0.0/16 => (direct) on interface 0
DEBUG: adding route 10.0.0.0/8 => (direct) on interface 1
DEBUG: adding route 172.16.0.0/12 => 10.0.0.172 on interface 1
DEBUG: Network interface has Ethernet address aa:2d:38:87:c1:84 and IP address 192.168.0.50
DEBUG: Connecting from 192.168.0.50:34883...
DEBUG: Connecting to 172.16.0.100:1234...
Successfully connected to 172.16.0.100:1234.
hello i am the server
 
hello i am the client 

```

### 传送文件

实验中使用哈希工具来验证文件的发送和接受是否一致，结果如下，可以看到文件在本设备的两个进程之间成功传送

```commandline
cs144@vm:/tmp$ sha256sum /tmp/big.txt
e2ecfe84ff4d3e0944886d3dfd631503ab6e1fc7022afc9d70a3c07a35e0d26f  /tmp/big.txt
cs144@vm:/tmp$ sha256sum /tmp/big-received.txt
e2ecfe84ff4d3e0944886d3dfd631503ab6e1fc7022afc9d70a3c07a35e0d26f  /tmp/big-received.txt
```