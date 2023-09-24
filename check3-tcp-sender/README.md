# Check3 Notes

## Introduction

There are three main responsibilities for the TCP sender:
- Handle acknowledgments received from the receiver, continuously track the sliding window size and the starting position of the window.
- Read bytes from the sender's `ByteStream` and attempt to fill the sliding window.
- Track packets that have been sent but not yet acknowledged by the receiver (these packets may be lost).

You need to implement these functionalities in this experiment.

## Hint

### How to implement a timer?

Here I'll share my approach, which is not complicated. Don't get overwhelmed by the complexity of object-oriented programming.

```c++
//! Retransmission timer for the sender
class Timer
{
private:
  //! Elapsed time
  uint64_t _time_count = 0;
  //! Predefined countdown
  uint64_t _time_out = 0;
  //! Indicates if a timeout has occurred
  bool _is_timeout = false;
  //! Whether the timer is active
  bool _active = false;

public:
  Timer() = default;
  Timer(uint64_t init) { _time_out = init; }
  void tick(const uint64_t ms_since_last_tick);
  //! Check if the timer has timed out
  bool is_time_out() const { return _is_timeout; }
  //! Reset the timer
  void restart();
  //! Set the countdown time
  void set_time_out(uint64_t time_out) { _time_out = time_out; }
  //! Deactivate the timer
  void disactive() { _active = false; }
  bool is_active() const { return _active; }
  uint64_t get_doubled_time_out() const { return 2 * _time_out; }
};
```

### Timeout

After reading the requirements, you might attempt to create a data structure to track every sent but unacknowledged data packet. You may try to assign a Timer to each data packet.

This is unnecessary. This experiment expects you to only track and retransmit the oldest (expired) data packet. So, the Timer is a singleton. This point needs to be clear, or it can cause significant confusion.

I've hinted at the appropriate data structure here. Heap? Or queue? Think carefully!

### window_size and SYN

`window_size` includes SYN, so when calculating the data length, subtract the possible presence of the SYN bit.

### What is continuous retransmission count, and how do I update it?

The parameter `_consecutive_retransmissions_count` that records the number of consecutive retransmissions is for testing purposes. To understand how to update this parameter as the experiment author expects, you need to carefully read the test cases.

This is unrelated to the functionality of TCP itself.

Test results:

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
      Start 31:

 send_window
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