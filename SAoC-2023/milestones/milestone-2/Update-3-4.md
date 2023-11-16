# [SAoC 2023] Replace DRuntime Hooks with Templates Weekly Update #7-8

Hi,

Last week I fixed [this bug](https://issues.dlang.org/show_bug.cgi?id=24159) introduced by `_d_arrayappendcTX`.
The fix meant first storing the lowering to a new field inside `CatAssignExp` instead of modifying the original array.

This in turn caused `vthis`s for some functions because of the inliner.
The solution was to [inline the lowering](https://github.com/teodutu/dmd/blob/71518bee8b2779a9cd6064537470582dae4c29c1/compiler/src/dmd/optimize.d#L931-L937) when present.

After this, I ran into a familiar error: the lowering needs no codegen at CTFE.
In fact, the lowering shouldn't even be introduced during semantic if the `CatAssignExp` is only needed at CTFE.
However, this problem is familiar because I ran into it in the past.
The same restrictions hold true for `CatExp`s, but because telling if an expression needs codegen or not is a mess during semantic, I couldn't figure out when not to perform the lowering to `_d_arraycatnTX`.
The solution in both cases was to handle this case during IR generation.
Therefore, unfortunately, both lowerings are made even in some situations when they're not required and then `e2ir.d` checks of using the GC is enabled.
If not, [it outputs an error](https://github.com/dlang/dmd/blob/3d552df287d0b836861f760701b16569311e4dd7/compiler/src/dmd/e2ir.d#L2796-L2802).
I'd love to do this in a nicer way in the future and I'm looking for advice on how to figure out when to not generate lowerings during semantic analysis.
If you have any advice on the topic, please let me know.

Furthermore, I'm almost done converting `_d_newarray{mTX, miTX, Op}` to a single template.
I created [this draft PR](https://github.com/dlang/dmd/pull/15819) with my changes.
I still need to fix some final tests in dmd and Phobos before turning it into a legit PR.
I need to say that the previous implementations of `_d_newarraym{,i}TX` were really messy because they called `_d_newarrayOp` with a funcion argument with which to initialise the arrays (one of `_d_newarray{,i}T`).
But now that [I've converted these 2 to a single template](https://github.com/dlang/dmd/pull/15299) as well, I was able to coalesce all hooks for multi-dimensional arrays into a single one much more smoothly.

Thanks,\
Teodor
