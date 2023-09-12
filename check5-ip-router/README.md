# Check5 Notes

## Introduction

This experiment guides you through implementing basic router **forwarding** functionality, helping you comprehend the working principles of routing tables and the **longest prefix matching mechanism**.

## Hint

### Next Hop in the Routing Table

The next hop can be empty, indicating that the router (or device) is directly connected to that network segment (even in this case, the port number will still be stored in the table entry). The routing table can have a maximum of four entries and a minimum of three entries.

**The next hop address is used to assist the ARP protocol in populating the destination MAC address of the second-layer Ethernet frame and is not used to modify IP data packets.**

### Longest Prefix Match

The principle of longest prefix matching involves selecting the table entry with the longest prefix length among all the routing table entries that match the mask and destination address obtained by performing a bitwise AND operation.

```c++
auto max_prefix_it = routeTable_.end();
for (auto it = routeTable_.begin(); it != routeTable_.end(); ++it) {
    if (((uint64_t)dst_) >> (32 - it->prefix_length) == (((uint64_t)it->route_prefix) >> (32 - it->prefix_length))) {
        // match
        if (max_prefix_it == routeTable_.end() || max_prefix_it->prefix_length < it->prefix_length) {
            max_prefix_it = it;
        }
    }
}
```

It's worth noting that in C/C++, when you right-shift a value by an amount equal to its bit length, it will perform a modulo operation.

```c++
#include <iostream>
#include <string>
#include <list>
using namespace std;
int main(int argc, char *argv[]) {
    unsigned int x = 0b11111111111111111111111111111111;
    cout << x << endl;
    unsigned int k = x >> 32;
    cout << k << endl;
    unsigned int y = x >> 31;
    cout << y << endl;
    unsigned int z = x >> 5;
    cout << z << endl;
    return 0;
}
```

The code above produces the following results. If you right-shift a 32-bit unsigned integer by 32 bits, the **original value remains unchanged**. It's important to be aware of this behavior when writing code. To address this, you can cast the data type to 64 bits before performing bitwise operations.

```commandline
4294967295
4294967295
1
134217727

Process finished with exit code 0
```

### TTL (Time-to-Live)

When forwarding, it's necessary to check the TTL field and decrement it. The subsequent checksum will verify this TTL.

### IP Packet Checksum (IPv4 Header Checksum)

If you don't recalculate the checksum of the IP packet when forwarding it, the receiving party (next hop) will detect an abnormal packet when parsing the received IP data packet because the checksum covers the entire IPv4 header.

[https://en.wikipedia.org/wiki/Internet_checksum](https://en.wikipedia.org/wiki/Internet_checksum)

> If there is no corruption, the result of summing the entire IP header, including checksum, should be zero. At each hop, the checksum is verified. Packets with checksum mismatch are discarded. The router must adjust the checksum if it changes the IP header (such as when decrementing the TTL).

```commandline
cs144@vm:~/minnow$ cmake --build build --target check5
Test project /home/cs144/minnow/build
    Start  1: compile with bug-checkers
1/3 Test  #1: compile with bug-checkers ........   Passed   18.78 sec
    Start 35: net_interface
2/3 Test #35: net_interface ....................   Passed    0.08 sec
    Start 36: router
3/3 Test #36: router ...........................***Failed    0.07 sec
Constructing network.
DEBUG: Network interface has Ethernet address 02:00:00:83:d9:c0 and IP address 171.67.76.46
DEBUG: Network interface has Ethernet address 02:00:00:4e:b0:83 and IP address 10.0.0.1
DEBUG: Network interface has Ethernet address 02:00:00:25:f4:25 and IP address 172.16.0.1
DEBUG: Network interface has Ethernet address 02:00:00:95:da:6e and IP address 192.168.0.1
DEBUG: Network interface has Ethernet address 02:00:00:fd:86:bf and IP address 198.178.229.1
DEBUG: Network interface has Ethernet address 02:00:00:32:f0:2d and IP address 143.195.0.2
DEBUG: Network interface has Ethernet address 02:00:00:55:e0:44 and IP address 128.30.76.255
DEBUG: Network interface has Ethernet address 6a:d3:88:15:6a:2a and IP address 10.0.0.2
DEBUG: Network interface has Ethernet address b2:dc:3f:74:f6:a6 and IP address 171.67.76.1
DEBUG: Network interface has Ethernet address b2:f4:74:96:6c:fd and IP address 192.168.0.2
DEBUG: Network interface has Ethernet address 2e:99:a8:4e:4d:db and IP address 143.195.0.1
DEBUG: Network interface has Ethernet address 6a:59:24:09:59:4a and IP address 198.178.229.42
DEBUG: Network interface has Ethernet address 7a:52:46:75:ec:88 and IP address 198.178.229.43
DEBUG: adding route 0.0.0.0/0 => 171.67.76.1 on interface 0
DEBUG: adding route 10.0.0.0/8 => (direct) on interface 1
DEBUG: adding route 172.16.0.0/16 => (direct) on interface 2
DEBUG: adding route 192.168.0.0/24 => (direct) on interface 3
DEBUG: adding route 198.178.229.0/24 => (direct) on interface 4
DEBUG: adding route 143.195.0.0/17 => 143.195.0.1 on interface 5
DEBUG: adding route 143.195.128.0/18 => 143.195.0.1 on interface 5
DEBUG: adding route 143.195.192.0/19 => 143.195.0.1 on interface 5
DEBUG: adding route 128.30.76.255/16 => 128.30.0.1 on interface 6


Testing traffic between two ordinary hosts (applesauce to cherrypie)...

Host applesauce trying to send datagram (with next hop = 10.0.0.1): IPv4, len=30, protocol=6, src=10.0.0.2, dst=192.168.0.2 payload="random payload: {2485243765}"
Transferring frame from applesauce to router.eth0: dst=ff:ff:ff:ff:ff:ff, src=6a:d3:88:15:6a:2a, type=ARP, payload: ARP: opcode=REQUEST, sender=6a:d3:88:15:6a:2a/10.0.0.2, target=00:00:00:00:00:00/10.0.0.1
Transferring frame from router.eth0 to applesauce: dst=6a:d3:88:15:6a:2a, src=02:00:00:4e:b0:83, type=ARP, payload: ARP: opcode=REPLY, sender=02:00:00:4e:b0:83/10.0.0.1, target=6a:d3:88:15:6a:2a/10.0.0.2
Transferring frame from applesauce to router.eth0: dst=02:00:00:4e:b0:83, src=6a:d3:88:15:6a:2a, type=IPv4, payload: IPv4: IPv4, len=30, protocol=6, src=10.0.0.2, dst=192.168.0.2 payload="random payload: {2485243765}"
Transferring frame from router.eth2 to cherrypie: dst=ff:ff:ff:ff:ff:ff, src=02:00:00:95:da:6e, type=ARP, payload: ARP: opcode=REQUEST, sender=02:00:00:95:da:6e/192.168.0.1, target=00:00:00:00:00:00/192.168.0.2
Transferring frame from cherrypie to router.eth2: dst=02:00:00:95:da:6e, src=b2:f4:74:96:6c:fd, type=ARP, payload: ARP: opcode=REPLY, sender=b2:f4:74:96:6c:fd/192.168.0.2, target=02:00:00:95:da:6e/192.168.0.1
Transferring frame from router.eth2 to cherrypie: dst=b2:f4:74:96:6c:fd, src=02:00:00:95:da:6e, type=IPv4, payload: bad IPv4 datagram



Error: Host cherrypie did NOT receive an expected Internet datagram: IPv4, len=30, protocol=6, src=10.0.0.2, dst=192.168.0.2


67% tests passed, 1 tests failed out of 3

Total Test time (real) =  18.94 sec

The following tests FAILED:
         36 - router (Failed)
Errors while running CTest
make[3]: *** [CMakeFiles/check5.dir/build.make:70: CMakeFiles/check5] Error 8
make[2]: *** [CMakeFiles/Makefile2:3588: CMakeFiles/check5.dir/all] Error 2
make[1]: *** [CMakeFiles/Makefile2:3595: CMakeFiles/check5.dir/rule] Error 2
make: *** [Makefile:1707: check5] Error 2
```

Added checksum method, test successful:

```commandline
cs144@vm:~/minnow$ cmake --build build --target check5
Test project /home/cs144/minnow/build
    Start  1: compile with bug-checkers
1/3 Test  #1: compile with bug-checkers ........   Passed   12.57 sec
    Start 35: net_interface
2/3 Test #35: net_interface ....................   Passed    0.07 sec
    Start 36: router
3/3 Test #36: router ...........................   Passed    0.13 sec

100% tests passed, 0 tests failed out of 3

Total Test time (real) =  12.78 sec
Built target check5
```