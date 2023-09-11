# Check4 Notes

## Introduction

本实验的目的旨在实现第三层和第二层网络之间的接口，这是计算机网络中网络层之下的部分：（bridges the worlds of Internet datagrams and of link-layer
frames）

1. IP报文封装成以太网帧，转发到链路；收到以太网帧后进行解析
2. ARP Request 和 ARP Reply

因为路由器等网络设备也包含第三层和第二层网络，所以这个模块也会被**路由器**使用

本实验并不涉及RARP协议

## Hint

### 接口模块所需要维护的几个缓存

合理的接口设计至少需要同时在内存中维护**三个缓存数据结构**

#### Cache1: ARP Table

该缓存（表）负责维护32位IP地址到48位MAC地址之间的映射关系

以太网帧在发送之前需要在其帧头部填写相应的接受MAC地址，以太网使用的是MAC地址而不是第三层网络层所使用的IPv4逻辑地址

缓存实现：也许你会试图创建自定义的哈希函数来哈希`Address`类（你担心通过`ipv4_numeric()`方法获得的32位整数类型无法表达`Address`类的全部信息）：

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

这是**不必要**的，因为ARP Table**不需要IP地址的端口信息**，他只需要32位IP地址，所以合适的做法是将哈希表的key设置为`uint32_t`

```c++
std::unordered_map<uint32_t, std::pair<EthernetAddress, uint64_t>> arp_table{};
```

#### Cache2: Arp Reply

如果ARP Table中的表项**过期或是缺失**，则需要使用ARP协议来和目标设备进行通讯，以获悉目标的新MAC地址，从而更新ARP表

ARP协议的发送方在ARP Request发送完毕后，需要等待ARP Reply。如果迟迟无法收到目标设备的ARP Reply以太网帧，则重新发送ARP Request

这个缓存就是**用来记录各个ARP Request的发送时间戳**的。如果请求超时，则重发

缓存实现：使用哈希表建立IP地址到该ARP Request的TTL之间的映射：

```c++
std::unordered_map<uint32_t, uint64_t> wait_arp_reply{};
```

#### Cache3: IP Packets Wait to be Sent

ARP Request发送时机一般是在需要发送IP packet但ARP Table查无此项时。我们**想要发送但却暂时无法发送的IP数据包**需要放入这个缓存中，当我们收到某一个ARP Reply的时候，对这个缓存进行清理

缓存实现：哈希表配合序列化容器，建立目标IP地址到IP包的一对多映射：（你当然可以不用`std::vector`，事实上`std::list`或者`std::forward_list`对内存更友好）

```c++
std::unordered_map<uint32_t, std::vector<InternetDatagram>> ipDatagram_wait_to_send{};
```

当我们在收到ARP Reply的时候，只需要查出对应的向量然后将向量中的元素逐一发送即可：

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

#### 更新缓存的时机

实验只要求我们跟踪ARP表表项和Arp Request的TTL，也就是说当`tick()`被调用的时候，你只需要更新这两个Cache即可

但实际的实现中，我们同样需要跟踪**IP Packets Wait to be Sent**这个缓存的TTL，如果超时，则丢弃（best-effort service的网络层）

此外在实际情况下，如果**Arp Reply**中的请求TTL到期，我们不仅需要重发**ARP Request**，还需要使用**ICMP**向目标对象发送**host unreachable**的异常

测试结果：

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