# Check1笔记

## 简介

Check1的任务是在Check0设计好的`ByteStream`组件基础上实现TCP的组装器`Reassembler`。组装器的任务是将收到的带有`offset`的字符串按照顺序塞入`ByteStream`的一端。

此外，组装器有和ByteStream**共享的内存空间**，可以暂存无法即刻塞入`ByteStream`的字符串。使用共享内存容量的目的是便于控制这两个组件的总内存开销。

我认为Check1的难点是你需要使用C++选择并设计合理的数据结构，高效地管理`Reassembler`内存。收纳暂存的字符串并在合适的时机清理不需要的字符串碎片的逻辑比较棘手。

## Hint

官方的测试样例似乎是全覆盖的，我尝试过修改和增加测试样例以增加测试覆盖度。你也可以像我一样做，这并不难。

官方给出的测试样例有一定的随机性。**就算程序通过了截止到`reassembler_win`的全部测试样例，你的程序依然有可能存在逻辑错误（且很难debug）**。你会发现`reassembler_speed_test`无法通过测试。

在这种情况下你需要重新思考程序的逻辑，而不是过度依赖测试样例的输出结果。我曾经在`reassembler_speed_test`源代码中大改一番，反复编译测试用例，企图在随机数中寻找规律，但这么做收效甚微。最快的做法是重新审视你的程序。

## 如何优化速度

TCP的稳定性很重要，但执行速度和benchmark结果同样理应是TCP开发的关注重点（**网络通讯协议，速度始终是重要指标和优化对象**）。

你可能在一开始希望通过建立哈希表（C++的`std::unordered_map`）来存储无法立即塞入管道的`string`，因为哈希表很快，我们一般倾向于认为哈希表是常数时间复杂度。

但不幸的是，测试样例很快暗示你必须手动管理字典中的`string`碎片。这意味着需要对`string`的`offset`做排序。所以还是选择用`O(logN)`的`std::map`吧。

如果你很有野心，你也可以试试用`std::array`或是C风格的数组配合指针来保存字符串，这将是十分激进的优化策略。

`std::map`迭代器的操作需要熟练，否则开发过程会变得艰难。我的程序涉及到同时维护和迭代两个指向同一个`std::map`的迭代器。注意是一个双向迭代器，且`insert`和`erase`操作都会造成迭代器失效。

我的程序还涉及`key`查找的步骤，我尝试使用`map::lower_bound()`来快速定位`offset`。这是一个Logarithmic in the size of the container复杂度的算法。对数级别的时间复杂度将比遍历整张`std::map`要来的快。

benchmark结果显示我的实现版本来到了6.73 Gbit/s，和官方给出的最佳速度10 Gbit/s比较接近，也许使用`std::array`会更快。

```shell
cs144@vm:~/minnow$ cmake --build build --target check1
Test project /home/cs144/minnow/build
      Start  1: compile with bug-checkers
 1/17 Test  #1: compile with bug-checkers ........   Passed    0.79 sec
      Start  3: byte_stream_basics
 2/17 Test  #3: byte_stream_basics ...............   Passed    0.03 sec
      Start  4: byte_stream_capacity
 3/17 Test  #4: byte_stream_capacity .............   Passed    0.04 sec
      Start  5: byte_stream_one_write
 4/17 Test  #5: byte_stream_one_write ............   Passed    0.04 sec
      Start  6: byte_stream_two_writes
 5/17 Test  #6: byte_stream_two_writes ...........   Passed    0.04 sec
      Start  7: byte_stream_many_writes
 6/17 Test  #7: byte_stream_many_writes ..........   Passed    0.09 sec
      Start  8: byte_stream_stress_test
 7/17 Test  #8: byte_stream_stress_test ..........   Passed    0.04 sec
      Start  9: reassembler_single
 8/17 Test  #9: reassembler_single ...............   Passed    0.09 sec
      Start 10: reassembler_cap
 9/17 Test #10: reassembler_cap ..................   Passed    0.03 sec
      Start 11: reassembler_seq
10/17 Test #11: reassembler_seq ..................   Passed    0.04 sec
      Start 12: reassembler_dup
11/17 Test #12: reassembler_dup ..................   Passed    0.07 sec
      Start 13: reassembler_holes
12/17 Test #13: reassembler_holes ................   Passed    0.03 sec
      Start 14: reassembler_overlapping
13/17 Test #14: reassembler_overlapping ..........   Passed    0.03 sec
      Start 15: reassembler_win
14/17 Test #15: reassembler_win ..................   Passed    0.50 sec
      Start 16: compile with optimization
15/17 Test #16: compile with optimization ........   Passed    0.24 sec
      Start 17: byte_stream_speed_test
             ByteStream throughput: 0.64 Gbit/s
16/17 Test #17: byte_stream_speed_test ...........   Passed    0.41 sec
      Start 18: reassembler_speed_test
             Reassembler throughput: 6.73 Gbit/s
17/17 Test #18: reassembler_speed_test ...........   Passed    0.47 sec

100% tests passed, 0 tests failed out of 17

Total Test time (real) =   2.99 sec
Built target check1
```