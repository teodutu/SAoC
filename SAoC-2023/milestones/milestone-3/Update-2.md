# [SAoC 2023] Replace DRuntime Hooks with Templates Weekly Update #10

Hi,

Last week I started to convert [`_d_arrayliteralTX`](https://github.com/dlang/dmd/blob/41f3577d7d8f63a7221a9ff2359fe4e5163643cd/druntime/src/rt/lifetime.d#L2099-L2140) to a template.
The hook is somewhat similar to [`_d_newarrayT`](https://github.com/dlang/dmd/pull/15299) in that it's used to allocate array literals like so `[1, 2, 3]`.
Note that unlike `_d_newarrayT`, which also initialises the newly created array, `_d_arrayliteralTX` just allocates memory for it.
The reason for this choice is to limit the number of arguments passed to the hook.
If it were to also initialise the array, it would have to receive its contents as an extra variadic argument.
I am going to stick to this decision because now with the template hook, an extra variadic template argument would mean a lot more instances for something that is already handled just as fine and with less code by the compiler.

One annoying fact about array literals is that they suffer lots of optimisations after semantic analysis (so after the introduction of the hook).
In short, they are either eliminated altogether, or coalesced with concatenations into larger literals.
For example, `[1, 2, 3] ~ 4` is optimised by the compiler to `[1, 2, 3, 4]`.
This becomes a nuissance because the hook receives the length of the array as an argument and this length changes in the previous example.
Since the lowering is introduced during [semantic analysis](https://github.com/dlang/dmd/blob/41f3577d7d8f63a7221a9ff2359fe4e5163643cd/compiler/src/dmd/expressionsem.d#L4243) and this optimisation (which is not the only one that may happen) takes place during [optimize.d, so after semantic analysis](https://github.com/dlang/dmd/blob/41f3577d7d8f63a7221a9ff2359fe4e5163643cd/compiler/src/dmd/optimize.d#L1288), I'd have to make changes to `ArrayLiteralExp`s in multiple places, which does not scale and will introduce bugs if another change to `ArrayLiteralExp` is introduced without taking the hook into account.

A cleaner solution which I'm going to try this week is to inspect the call to the hook once more during [IR generation](https://github.com/dlang/dmd/blob/41f3577d7d8f63a7221a9ff2359fe4e5163643cd/compiler/src/dmd/e2ir.d#L3957) and update the array length argument according to the new size of the array.

Thanks,\
Teodor
