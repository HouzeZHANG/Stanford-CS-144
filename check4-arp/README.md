# Check4 Notes

## Introduction

The purpose of this experiment is to implement the interface between the third and second layers of the network, which is the portion below the network layer in computer networks: (bridges the worlds of Internet datagrams and of link-layer frames).

1. Encapsulation of IP packets into Ethernet frames and forwarding them to the link; parsing Ethernet frames upon receipt.
2. Handling ARP Request and ARP Reply.

Because routers and other network devices also involve the third and second layers of the network, this module will also be used by routers.

This experiment **does not involve** the RARP protocol.

## Hint

### Several Caches to be Maintained by the Interface Module

A well-designed interface module needs to maintain at least **three cache data structures** simultaneously in memory.

#### Cache1: ARP Table

This cache (table) is responsible for maintaining the mapping between 32-bit IP addresses and 48-bit MAC addresses.

Ethernet frames need to fill in the corresponding destination MAC address in their frame headers before sending. Ethernet uses MAC addresses, not the IPv4 logical addresses used by the third layer network.

Cache implementation: You may attempt to create a custom hash function to hash the `Address` class (if you are concerned that the 32-bit integer obtained through the `ipv4_numeric()` method cannot represent all the information of the `Address` class):

```c++
std::hash<std::string> str_hasher;

template<>
struct std::hash<Address>
{
    std::size_t operator()(const Address& address) const{
        return str_hasher(address.to_string());
    }
};
```

This is **unnecessary** because the ARP Table **does not need the port information of IP addresses**; it only needs the 32-bit IP address. Therefore, the appropriate approach is to set the key of the hash table as `uint32_t`:

```c++
std::unordered_map<uint32_t, std::pair<EthernetAddress, uint64_t>> arp_table{};
```

#### Cache2: ARP Reply

If an entry in the ARP Table is **expired or missing**, the ARP protocol needs to be used to communicate with the target device to obtain the new MAC address of the target and update the ARP table.

The sender of the ARP protocol needs to wait for an ARP Reply after sending an ARP Request. If the ARP Reply from the target device is not received promptly, the ARP Request needs to be resent.

This cache is used to record the timestamps of various ARP Requests. If a request times out, it is resent.

Cache implementation: Use a hash table to establish a mapping between IP addresses and the TTL of this ARP Request:

```c++
std::unordered_map<uint32_t, uint64_t> wait_arp_reply{};
```

#### Cache3: IP Packets Wait to be Sent

The timing for sending ARP Requests is generally when an IP packet needs to be sent, but there is no corresponding entry in the ARP Table. IP packets that we **want to send but cannot send temporarily** should be placed in this cache. When we receive an ARP Reply, we can clean up this cache.

Cache implementation: Use a hash table in combination with a serialized container to establish a one-to-many mapping between the target IP address and IP packets. (You can certainly use `std::vector`; in fact, `std::list` or `std::forward_list` is more memory-friendly):

```c++
std::unordered_map<uint32_t, std::vector<InternetDatagram>> ipDatagram_wait_to_send{};
```

When we receive an ARP Reply, we only need to retrieve the corresponding vector and send the elements one by one:

```c++
} else if ( arpRecv.opcode == ARPMessage::OPCODE_REPLY ) {
	//! arpReply
	if ( arpRecv.target_ip_address == ip_address_.ipv4_numeric() ) {
		//! reply to me
		auto sender_ip = arpRecv.sender_ip_address;
		if ( ipDatagram_wait_to_send.find( sender_ip ) != ipDatagram_wait_to_send.end() ) {
		auto& wait_to_send_vec = ipDatagram_wait_to_send[sender_ip];
		for ( const auto& msg : wait_to_send_vec ) {
			send_datagram( msg, Address::from_ipv4_numeric( sender_ip ) );
		}
			ipDatagram_wait_to_send.erase( sender_ip );
		}
	}
}
```

#### Cache Update Timing

The experiment only requires us to track ARP table entries and the TTL of ARP Requests, which means that when `tick()` is called, you only need to update these two caches.

However, in practice, we also need to track the TTL of the **IP Packets Wait to be Sent** cache. If it times out, it should be discarded (network layer of best-effort service).

In addition, in practical situations, if the TTL of an ARP Reply request expires, we need to not only resend the ARP Request but also use ICMP to send a **host unreachable** exception to the target object.

Test results:

```shell
cs144@vm:~/minnow$ cmake --build build --target check4
Test project /home/cs144/minnow/build
    Start  1: compile with bug-checkers
1/2 Test  #1: compile with bug-checkers ........   Passed    1.28 sec
    Start 35: net_interface
2/2 Test #35: net_interface ....................   Passed    0.04 sec

100% tests passed, 0 tests failed out of 2

Total Test time (real) =   1.33 sec
Built target check4
```