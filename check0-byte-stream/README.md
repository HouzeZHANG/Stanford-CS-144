# Check1 Notes

## Introduction

**check0** is an experiment about byte streams. This experiment helps you set up a basic experimental environment, familiarize yourself with the development process, and submission process.

This experiment helps you understand the **byte stream abstraction** of the TCP/IP protocol stack and guides you to design and implement a `wget` program using C++ and the `Socket` interface, as well as a reliable **single-machine byte stream abstraction**.

## Environment Setup

Official instructions: [https://stanford.edu/class/cs144/vm_howto/](https://stanford.edu/class/cs144/vm_howto/)

The environment I chose is Windows 11 + VirtualBox + Linux VM (official virtual machine template).

Development environment: Host machine with **VS Code remote SSH** connection to the local virtual machine, development using VS Code, debugging using **GDB**.

The virtual machine is configured with 8GB of memory and 4 cores, and this virtual machine's configuration will affect the benchmark results of the programs you develop later.

### Hint

Please **do not attempt to install the Wakatime plugin** on the virtual machine to track your coding time, as this may cause your virtual machine to crash (I have actually tried this).

Please do not attempt to create multiple virtual machines simultaneously, as the official virtual machine image prohibits creating multiple virtual machines (I suspect this is due to port forwarding and resource usage issues, as the default configuration of this virtual machine image occupies port 2222 on the host machine).

Due to the vulnerabilities in Linux, please use VCS to synchronize your work to a remote VCS server after completing a module. I chose **Github** for this purpose.

## Exercise 1: Fetch Web Page And Send Email

### Hint

When using `telnet` to establish an `HTTP` connection with the server, if the request is not sent quickly enough, the server may return a timeout error report, so please type quickly. We noticed that `408` is a timeout indicator.

```shell
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>408 Request Timeout</title>
</head><body>
<h1>Request Timeout</h1>
<p>Server timeout waiting for the HTTP request from the client.</p>
</body></html>
```

The following is the correct response:

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

I am not currently a Stanford student, so I do not have a Stanford Email account. I tried using `telnet` to connect to other mail servers. Please note that not all mail servers have the `smtp` service open, this depends on the configuration of the mail server. For example, UTC's mail server only supports `smtps`, while Tencent's `QQ mail` supports both.

First, make sure you can `ping` the school's email sending server.

```shell
cs144@vm:~/lab0$ ping smtps.utc.fr
PING smtps.utc.fr (195.83.155.8) 56(84) bytes of data.
64 bytes from smtps.utc.fr (195.83.155.8): icmp_seq=1 ttl=46 time=70.8 ms
64 bytes from smtps.utc.fr (195.83.155.8): icmp_seq=2 ttl=46 time=66.8 ms
64 bytes from smtps.utc.fr (195.83.155.8): icmp_seq=3 ttl=46 time=54.4 ms
```

Our school's mail server can only be used with `smtps`.

```shell
cs144@vm:~/lab0$ telnet 195.83.155.8 smtps
Trying 195.83.155.8...
Connected to 195.83.155.8.
Escape character is '^]'.
```


Try with a `QQ mail` mailbox.

```shell
cs144@vm:~/lab0$ ping smtp.qq.com
PING smtp.qq.com (43.129.255.54) 56(84) bytes of data.
64 bytes from 43.129.255.54 (43.129.255.54): icmp_seq=1 ttl=47 time=294 ms
64 bytes from 43.129.255.54 (43.129.255.54): icmp_seq=2 ttl=47 time=280 ms
64 bytes from 43.129.255.54 (43.129.255.54): icmp_seq=3 ttl=47 time=289 ms
```

It worked, both `smtp` and `smtps` are supported by the `QQ mail` server.

```shell
cs144@vm:~/lab0$ telnet 43.129.255.54 smtp
Trying 43.129.255.54...
Connected to 43.129.255.54.
Escape character is '^]'.
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

## Exercise 2: `natcat`

A very simple experiment, not much to explain.

## Exercise 3: Using `stream socket` to Write a Network Program

The theme of this experiment is to use `Socket` to simulate the functionality of accessing a server and downloading a web page using `telnet` on the command line in C++.

### Hint

When compiling, you need to enter the `build` directory before compiling. You can modify the test cases, but please do not modify the `CMakeLists.txt` file without authorization.

The last sentence of the request message needs to include two sets of newline characters.

```C++
const string request = "GET " + path + " HTTP/1.1\r\n" + 
    "Host: " + host + "\r\n" + 
    "Connection: close\r\n\r\n";
```

Before writing the program, make sure you understand the concept of `string_view`. Incorrect use of `string_view` can result in garbled messages, and garbled request headers will cause the server to be unable to parse the request and return a `400 Bad Request` error.

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

The correct result is as follows, you should receive a `200 OK` response. Please do not add extra `\n` to `cout`, as this will cause the test to fail.

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

The correct test result:

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

## Exercise 4: Implement an In-Memory Reliable Byte Stream

When reading the requirements for this experiment, you may be puzzled by the characteristics of this component of TCP:

What is the significance of implementing a first-in, first-out byte stream queue on a single device? Also, you will notice that this queue does not support multi-threading concurrency. It does not support blocking reads and writes, and characters beyond the capacity will be discarded.

### Hint

Still, pay attention to `string_view`. I noticed that in the 2023 Spring, the return type of the `Reader::peek() const` interface changed from `string` to `string_view`, which is a new feature introduced in C++17.

Before designing your `ByteStream` class, make sure you understand the design philosophy of `string_view`.

The pitfall of `string_view` is that if you create a view for a local variable `string` (stack space), the string information retrieved from the `string_view` after the function call returns is undefined. The choice of the underlying data structure for `ByteStream` is crucial.

After the initial implementation, the benchmark results of my program were as follows:

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

I tried adjusting the construction of `string` using iterators, which reduced the program's execution time to about one-tenth of the previous time.

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
Exception: ByteStream did not meet the minimum speed of 0.1 Gbit/s.
```

It still did not meet the requirement, so I tried adjusting the underlying container to `string`, and the benchmark result was nearly ten times faster, successfully passing the speed test.

```shell
cs144@vm:~/minnow$ cmake --build build --target check0
Test project /home/cs144/minnow/build
      Start  1: compile with bug-checkers
 1/10 Test  #1: compile with bug-checkers ........   Passed    4.76 sec
      Start  2: t_webget
 2/10 Test  #

2: t_webget .........................   Passed    1.05 sec
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

With that, check0 comes to an end.