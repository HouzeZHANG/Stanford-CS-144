# Check1 Notes

## Introduction

The task in Check1 is to implement the TCP reassembler `Reassembler` based on the previously designed `ByteStream` component in Check0. The reassembler's job is to take received strings with offsets and assemble them in the correct order into one end of the `ByteStream`.

Additionally, the reassembler shares memory space with the `ByteStream` and can temporarily store strings that cannot be immediately placed into the `ByteStream`. Using shared memory capacity is essential for managing the overall memory overhead of these two components efficiently.

The main challenge in Check1 is selecting and designing appropriate data structures in C++, efficiently managing memory within the `Reassembler`, and implementing logic to store temporarily received strings and clean up unnecessary string fragments at the right time.

## Hint

The official test cases provided seem to have good coverage, but **you can try modifying and adding test cases to increase coverage further**. It's not a difficult task, and you can follow a similar approach as I did.

The official test cases have a certain level of **randomness**. Even if your program passes all test cases up to `reassembler_win`, there may still be logical errors in your program (and debugging them can be challenging). You will notice that `reassembler_speed_test` may not pass.

In such cases, you need to **reconsider the logic of your program** rather than relying too heavily on the output results of test cases. 

I once made significant changes to the `reassembler_speed_test` source code, repeatedly compiled test cases, and tried to find patterns in the random numbers, but this approach yielded limited results. **The fastest way is to reevaluate your program's logic**.

## How to Optimize for Speed

While TCP's reliability is crucial, execution speed and benchmark results are also important aspects of TCP development (network communication protocols inherently prioritize speed).

You may initially want to use a **hash table** (C++'s `std::unordered_map`) to store strings that cannot be immediately placed in the pipeline because hash tables are fast, and we generally assume constant time complexity.

Unfortunately, the test cases quickly suggest that you **must manually manage the fragments of strings in the dictionary**. This means you need to sort the `offset` of strings, so it's better to choose `std::map` with `O(logN)` complexity.

If you're ambitious, you can also try using `std::array` or C-style arrays **along with pointers to save strings**. This would be a more radical optimization strategy.

Working with `std::map` iterators can be tricky, as my program involved simultaneously **maintaining and iterating two iterators** pointing to the same `std::map`. Note that it's a **bidirectional iterator**, and both `insert` and `erase` operations can invalidate iterators.

My program also involved a step of looking up a `key`. I tried using `map::lower_bound()` to quickly locate the `offset`. This is an algorithm with **logarithmic complexity** in the size of the container. Logarithmic time complexity will be faster than traversing the entire `std::map`.

Benchmark results showed that my implementation version achieved **6.73 Gbit/s**, which is close to the best speed of 10 Gbit/s given by the official test cases. Using `std::array` might make it even faster.

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