# [SAoC 2021] Replace druntime Hooks with Templates: Milestone 3, Week 1

Hi,

This week I managed to fix the final bugs of the `_d_arrayctor` lowering.
In order to remove the call to `__ArrayDtor`, of which I spoke in [last week's post](https://forum.dlang.org/post/kocccsnqxjailweqcazz@forum.dlang.org), I resorted to declaring the array returned by `_d_arrayctor` inside a union.
This is somewhat unorthodox, but Adam Ruppe [assured me](https://forum.dlang.org/post/xqgfaicsqlbbekkbzqye@forum.dlang.org) that this is the accepted way of handling array destruction myself. 

As a result, this milestone I plan to run a benchmark for testing the performance increase when using the new lowering for `_d_array{,set}ctor`.
I am also looking to finish replacing the lowerings to `_d_arrayappendT` and `_d_arrayappendcTX` with templates.
I started working on this during the previous milestone and managed fixed a bug where CTFE did not correctly interpret the lowering to `_d_arrayappendcTX`.

After this, if time allows it, I'll move away for a while from lowerings involving array manipulation to hooks that manipulate somewhat simpler objects, such as `_d_delstruct`, `_d_newitem{T,iT,U}` or `_d_newThrowable`.
I plan to make this shift because I've seen array-based generate some rather time-consuming bugs and I hope that by doing so, I'll be able to get more work finished and get stuck less.

Thanks,\
Teodor
