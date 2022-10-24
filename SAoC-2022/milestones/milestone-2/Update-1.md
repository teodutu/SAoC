# [SAoC 2022] Replace druntime Hooks with Templates: Milestone 2, Week 1

Hi,

This week I continued with `_d_newitem{T,iT,U}` and `_d_arraycatnTX`.

I only implemented a template `_d_newitemU` and left the introduction of the call to `memset(0)` / `memcpy(T.init)` up to the compiler.
I am currently having trouble setting `vthis` for nested structs.
It is generally set in e2ir.d and reinventing this wheel in expressionsem.d is a bit trickier or I just don't know where to look.

I've fixed all easily reproducible bugs related to `_d_arraycatnTX` (PR [here](https://github.com/dlang/dmd/pull/14550)).
The ones that remain lie inside the compiler, unfortunately.
One of them was easy to track.
[Here](https://github.com/dlang/dmd/blob/9ecf81be49554550202fa61ab5d3ed68c2f37ce0/compiler/test/unit/lexer/location_offset.d#L556) 2 strings are concatenated and then the `.ptr` of the result is used.
Because of the missing `\0` at the end of strings, [this](https://github.com/dlang/dmd/blob/9ecf81be49554550202fa61ab5d3ed68c2f37ce0/compiler/test/unit/lexer/location_offset.d#L563) assert was failing.
The ctor of `Lexer` [says explicitly](https://github.com/dlang/dmd/blob/9ecf81be49554550202fa61ab5d3ed68c2f37ce0/compiler/src/dmd/lexer.d#L101) that its second argument should be null-terminated.
The solution was to manually add a `~ '\0'` at the end of the concatenation above.
I don't know how this error didn't manifest itself with the older hook.
My intuition says that the [remaining builkite failures](https://buildkite.com/dlang/dmd/builds/28559#0183fc10-644f-4cc9-a285-c96b76ee122a) also come from `.ptr` being used on strings, but I find it difficult to track them down.

Thanks,\
Teodor
