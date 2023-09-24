# Check3 Notes

## Introduction

TCP发送端有三个主要的职责：
- 处理收到的接收端的回报，实时跟踪滑动窗口大小和窗口的起始位置。
- 从发送端的`ByteStream`中读取字节，并试图填满滑动窗口。
- 跟踪已发送但未被接收端回报确认的包（这些包有可能丢失）

你在这个实验中需要完成实现这些功能。

## Hint

### 如何实现计时器？

这里分享一个我的思路，其实不复杂，不要被面向对象的繁琐所困扰。

```c++
//! 发送方的重传计时器
class Timer
{
private:
  //! 已经过去的时间
  uint64_t _time_count = 0;
  //! 被设定好的倒计时
  uint64_t _time_out = 0;
  //! 标识是否超时
  bool _is_timeout = false;
  //! 是否启动
  bool _active = false;

public:
  Timer() = default;
  Timer( uint64_t init ) { _time_out = init; }
  void tick( const uint64_t ms_since_last_tick );
  //! 判断计时器是否到期
  bool is_time_out() const { return _is_timeout; }
  //! 重置计时器
  void restart();
  //! 设置倒计时时间
  void set_time_out( uint64_t time_out ) { _time_out = time_out; }
  //! 关闭定时器
  void disactive() { _active = false; }
  bool is_active() const { return _active; }
  uint64_t get_doubled_time_out() const { return 2 * _time_out; }
};
```

### 超时

你可能在阅读要求后会试图创建一个数据结构来跟踪每一个已经发送但未被确认的数据包。你也许会试图给每一个数据包都配置一个Timer。

这是不必要的。这个实验希望你只跟踪并重传最旧的数据包（过期的）。所以Timer是单例。这一点需要明确，否则会给你带来不小的困扰。

我在这里已经给你暗示了合适的数据结构。堆？还是队列？仔细思考！

### window_size和SYN

`window_size`是包含SYN的，所以在计算数据长度的时候需要减去有可能存在的SYN位。

### 连续重传计数是什么？我该如何更新？

`_consecutive_retransmissions_count`这个记录连续重传个数的参数只为了测试。为了搞清楚实验作者希望你如何更新这个参数，你需要仔细阅读测试用例。

这和TCP的功能本身没有关系。

测试结果：

```commandline
cs144@vm:~/minnow$ cmake --build build --target check3
Test project /home/cs144/minnow/build
      Start  1: compile with bug-checkers
 1/36 Test  #1: compile with bug-checkers ........   Passed    1.46 sec
      Start  3: byte_stream_basics
 2/36 Test  #3: byte_stream_basics ...............   Passed    0.04 sec
      Start  4: byte_stream_capacity
 3/36 Test  #4: byte_stream_capacity .............   Passed    0.07 sec
      Start  5: byte_stream_one_write
 4/36 Test  #5: byte_stream_one_write ............   Passed    0.05 sec
      Start  6: byte_stream_two_writes
 5/36 Test  #6: byte_stream_two_writes ...........   Passed    0.04 sec
      Start  7: byte_stream_many_writes
 6/36 Test  #7: byte_stream_many_writes ..........   Passed    0.09 sec
      Start  8: byte_stream_stress_test
 7/36 Test  #8: byte_stream_stress_test ..........   Passed    0.04 sec
      Start  9: reassembler_single
 8/36 Test  #9: reassembler_single ...............   Passed    0.03 sec
      Start 10: reassembler_cap
 9/36 Test #10: reassembler_cap ..................   Passed    0.04 sec
      Start 11: reassembler_seq
10/36 Test #11: reassembler_seq ..................   Passed    0.04 sec
      Start 12: reassembler_dup
11/36 Test #12: reassembler_dup ..................   Passed    0.07 sec
      Start 13: reassembler_holes
12/36 Test #13: reassembler_holes ................   Passed    0.04 sec
      Start 14: reassembler_overlapping
13/36 Test #14: reassembler_overlapping ..........   Passed    0.04 sec
      Start 15: reassembler_win
14/36 Test #15: reassembler_win ..................   Passed    0.42 sec
      Start 16: wrapping_integers_cmp
15/36 Test #16: wrapping_integers_cmp ............   Passed    0.04 sec
      Start 17: wrapping_integers_wrap
16/36 Test #17: wrapping_integers_wrap ...........   Passed    0.02 sec
      Start 18: wrapping_integers_unwrap
17/36 Test #18: wrapping_integers_unwrap .........   Passed    0.02 sec
      Start 19: wrapping_integers_roundtrip
18/36 Test #19: wrapping_integers_roundtrip ......   Passed    0.95 sec
      Start 20: wrapping_integers_extra
19/36 Test #20: wrapping_integers_extra ..........   Passed    0.17 sec
      Start 21: recv_connect
20/36 Test #21: recv_connect .....................   Passed    0.06 sec
      Start 22: recv_transmit
21/36 Test #22: recv_transmit ....................   Passed    0.64 sec
      Start 23: recv_window
22/36 Test #23: recv_window ......................   Passed    0.08 sec
      Start 24: recv_reorder
23/36 Test #24: recv_reorder .....................   Passed    0.09 sec
      Start 25: recv_reorder_more
24/36 Test #25: recv_reorder_more ................   Passed    1.13 sec
      Start 26: recv_close
25/36 Test #26: recv_close .......................   Passed    0.05 sec
      Start 27: recv_special
26/36 Test #27: recv_special .....................   Passed    0.06 sec
      Start 28: send_connect
27/36 Test #28: send_connect .....................   Passed    0.05 sec
      Start 29: send_transmit
28/36 Test #29: send_transmit ....................   Passed    1.05 sec
      Start 30: send_retx
29/36 Test #30: send_retx ........................   Passed    0.08 sec
      Start 31: send_window
30/36 Test #31: send_window ......................   Passed    0.40 sec
      Start 32: send_ack
31/36 Test #32: send_ack .........................   Passed    0.07 sec
      Start 33: send_close
32/36 Test #33: send_close .......................   Passed    0.06 sec
      Start 34: send_extra
33/36 Test #34: send_extra .......................   Passed    0.13 sec
      Start 36: compile with optimization
34/36 Test #36: compile with optimization ........   Passed    7.70 sec
      Start 37: byte_stream_speed_test
             ByteStream throughput: 0.68 Gbit/s
35/36 Test #37: byte_stream_speed_test ...........   Passed    0.42 sec
      Start 38: reassembler_speed_test
             Reassembler throughput: 6.68 Gbit/s
36/36 Test #38: reassembler_speed_test ...........   Passed    0.59 sec

100% tests passed, 0 tests failed out of 36

Total Test time (real) =  16.41 sec
Built target check3
```