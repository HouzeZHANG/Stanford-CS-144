# Check2 Notes

## Introduction

TCP receiver 的主要职责有两个
- 将从对等体收到的信息装入组装器，这样组装器就能

TCP如何实现流量控制？
接收端维护的滑动窗口

sequence number

TCP ack and TCP flow control 是一对很关键的概念

TCP本身只能表示32位，但是流是无限长的。使用循环！

## Hint

### Wrap

思考如何将64位无符号整数转化32位无符号整数。

对无符号整数类型取后32位相当于对它取$2^32$模，

在interval == 0的时候要特殊处理，因为interval - 1会让left_candidate变得很大，

long long 的位数为64位，比较三个64位无符号整数类型的问题是比较棘手的
```c++
  return abs((long long)right_candidate - (long long)checkpoint) >= abs((long long)left_candidate - (long long)checkpoint) ? 
    left_candidate : 
    right_candidate;
```

在确保不是第一个interval的情况下，划分为三个区间，进行区间比较：
```c++
if (left_candidate >= checkpoint) {
	return left_candidate;
}
else if (right_candidate <= checkpoint) {
	return right_candidate;
}
else {
	if (checkpoint - left_candidate <= right_candidate - checkpoint) {
		return left_candidate;
	}

	return right_candidate;
}
```

SYN本身就占据了一个seq

```c++
      test.execute( SegmentArrives {}.with_syn().with_seqno( 0 ) );
      test.execute( ExpectAckno { Wrap32 { 1 } } );
```

```c++
  if (isn_set) {
    next = Wrap32::wrap(reassembler.get_ack_no(), isn) + 1;
  }
```

```optional?```

运行结果
```shell
cs144@vm:~/minnow$ cmake --build build --target check2
Test project /home/cs144/minnow/build
      Start  1: compile with bug-checkers
 1/29 Test  #1: compile with bug-checkers ........   Passed    2.65 sec
      Start  3: byte_stream_basics
 2/29 Test  #3: byte_stream_basics ...............   Passed    0.03 sec
      Start  4: byte_stream_capacity
 3/29 Test  #4: byte_stream_capacity .............   Passed    0.04 sec
      Start  5: byte_stream_one_write
 4/29 Test  #5: byte_stream_one_write ............   Passed    0.04 sec
      Start  6: byte_stream_two_writes
 5/29 Test  #6: byte_stream_two_writes ...........   Passed    0.03 sec
      Start  7: byte_stream_many_writes
 6/29 Test  #7: byte_stream_many_writes ..........   Passed    0.16 sec
      Start  8: byte_stream_stress_test
 7/29 Test  #8: byte_stream_stress_test ..........   Passed    0.06 sec
      Start  9: reassembler_single
 8/29 Test  #9: reassembler_single ...............   Passed    0.03 sec
      Start 10: reassembler_cap
 9/29 Test #10: reassembler_cap ..................   Passed    0.03 sec
      Start 11: reassembler_seq
10/29 Test #11: reassembler_seq ..................   Passed    0.04 sec
      Start 12: reassembler_dup
11/29 Test #12: reassembler_dup ..................   Passed    0.06 sec
      Start 13: reassembler_holes
12/29 Test #13: reassembler_holes ................   Passed    0.03 sec
      Start 14: reassembler_overlapping
13/29 Test #14: reassembler_overlapping ..........   Passed    0.03 sec
      Start 15: reassembler_win
14/29 Test #15: reassembler_win ..................   Passed    0.44 sec
      Start 16: wrapping_integers_cmp
15/29 Test #16: wrapping_integers_cmp ............   Passed    0.03 sec
      Start 17: wrapping_integers_wrap
16/29 Test #17: wrapping_integers_wrap ...........   Passed    0.02 sec
      Start 18: wrapping_integers_unwrap
17/29 Test #18: wrapping_integers_unwrap .........   Passed    0.03 sec
      Start 19: wrapping_integers_roundtrip
18/29 Test #19: wrapping_integers_roundtrip ......   Passed    1.19 sec
      Start 20: wrapping_integers_extra
19/29 Test #20: wrapping_integers_extra ..........   Passed    0.22 sec
      Start 21: recv_connect
20/29 Test #21: recv_connect .....................   Passed    0.03 sec
      Start 22: recv_transmit
21/29 Test #22: recv_transmit ....................   Passed    0.71 sec
      Start 23: recv_window
22/29 Test #23: recv_window ......................   Passed    0.02 sec
      Start 24: recv_reorder
23/29 Test #24: recv_reorder .....................   Passed    0.03 sec
      Start 25: recv_reorder_more
24/29 Test #25: recv_reorder_more ................   Passed    1.12 sec
      Start 26: recv_close
25/29 Test #26: recv_close .......................   Passed    0.03 sec
      Start 27: recv_special
26/29 Test #27: recv_special .....................   Passed    0.03 sec
      Start 28: compile with optimization
27/29 Test #28: compile with optimization ........   Passed    0.29 sec
      Start 29: byte_stream_speed_test
             ByteStream throughput: 0.57 Gbit/s
28/29 Test #29: byte_stream_speed_test ...........   Passed    0.43 sec
      Start 30: reassembler_speed_test
             Reassembler throughput: 5.36 Gbit/s
29/29 Test #30: reassembler_speed_test ...........   Passed    0.55 sec

100% tests passed, 0 tests failed out of 29

Total Test time (real) =   8.44 sec
Built target check2
```