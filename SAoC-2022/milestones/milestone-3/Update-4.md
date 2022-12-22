# [SAoC 2022] Replace druntime Hooks with Templates: Milestone 3, Week 4

Hi,

This week I haven't made much progress with `_d_arraycatnTX` and `_d_newitemT` due to some deadlines for university.
Hoping for a quicker PR, I started working on `_d_newclass`.
Similarly to `_d_newitemT`, this hook is used to lower `new C()` where `C` is a class.
I followed the same principle from the 2 hooks above: I added a new field to the `NewExp` class, called `lowering`.
During semantic analysis, the original `NewExp` is not removed like before, but this new field is set to the `CallExp` to `_d_newclass`.
This makes for fewer code changes in `e2ir.d` and makes setting the context pointer for nested classes easier since `e2ir.d` already does that.

Also similarly to `_d_newitem` and I am also close to finishing finishing this `_d_newclass`.
The only tests that fail now are some spurious errors that should only be printed for the original `NewExp`, but are now printed for the lowered expression as well.
One easy way to solve them would be to just gag all errors while running `expressionSemantic` on the lowering, but this risks leaking some ICEs, like the one in [this bug](https://issues.dlang.org/show_bug.cgi?id=22659) that I created a while ago.

Now that I'll get 2 weeks of holiday from university, I hope I'll have the time to finish these hooks and start working on `_d_newarray*`.

Merry Christmas!\
Teodor
