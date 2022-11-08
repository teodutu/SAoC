# [SAoC 2022] Replace druntime Hooks with Templates: Milestone 2, Weeks 2 & 3

Hi,

These weeks I finished implementing a template `_d_newitemU`.
It creates an uninitialised `struct` object.
It's used to lower expressions such as `new S(args)`, where `S` is a `struct`.

The compiler introduces the required `memset(p, 0, T.sizeof)` (for what was previously `_d_newitemT`) or `memcpy(p, T.init, T.sizeof)` (for what was previously `_d_newitemiT`) itself.
This helps reduce the number of templates defined in DRuntime in the attempt to "unify" some instantiations of `_d_newitemU`.

I am now working to get the final tests for `new S(...)` working, **where `S` is a non-nested `struct`**.
I haven't touched on nested structs yet as they require copying `vthis` and `vthis2`, which is cumbersome to do in `expressionsem.d`.
Currently, these fields are set in `e2ir.d`, where there is [existing machinery](https://github.com/dlang/dmd/blob/master/compiler/src/dmd/toir.d#L444-L484) for duplicating them.
I am now trying to "port" this code to `expressionsem.d`, but I'm struggling with this because there is no 1:1 mapping between the backend and frontend constructions.

Regarding [`_d_arraycatnTX`](https://github.com/dlang/dmd/pull/14550), I feel the work is _nearly_ done.
What's preventing buildkite to pass is that most likely there are cases in dmd where `.ptr` is used on strings that result from concatenations.
These strings don't have a `\0` character at the end, hence [the seg fault](https://buildkite.com/dlang/dmd/builds/28559#0183fc10-644f-4cc9-a285-c96b76ee122a/590-699).
I can't reproduce the issue on my machine, so I'm kind of left in the dark here.
Do you have any suggestions about how to find the cause of [this failure](https://buildkite.com/dlang/dmd/builds/28559#0183fc10-644f-4cc9-a285-c96b76ee122a)?

Thanks,\
Teodor
