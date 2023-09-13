# Check6 Notes

## Introduction

The purpose of this experiment is to assemble and verify the **TCP/IP protocol stack**.

To eliminate potential address translation issues caused by **NAT**, the experiment provides a dedicated **relay server** to connect two TCP hosts.

This experiment is straightforward and can be performed from the command line.

Based on the six components implemented earlier:

- Transport layer components: reassembler, bytestream, TCP-sender, TCP-receiver.

- IP layer: routing.

- And the IP/Ethernet Interface between the link layer and transport layer.

## Hint

### Port Number Increment

Please note that the client and server should specify **different** relay server ports to function correctly (X and X+1).

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

Let's try sending some bytes.

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

### File Transfer

In the experiment, a hash tool is used to verify whether the file sent and received match. The results are as follows, confirming the successful transfer of the file between two processes on this device.

```commandline
cs144@vm:/tmp$ sha256sum /tmp/big.txt
e2ecfe84ff4d3e0944886d3dfd631503ab6e1fc7022afc9d70a3c07a35e0d26f  /tmp/big.txt
cs144@vm:/tmp$ sha256sum /tmp/big-received.txt
e2ecfe84ff4d3e0944886d3dfd631503ab6e1fc7022afc9d70a3c07a35e0d26f  /tmp/big-received.txt
```