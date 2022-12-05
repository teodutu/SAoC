# [SAoC 2022] Replace druntime Hooks with Templates: Milestone 3, Week 3

Hi,

This week my mentors and I came up with a new lowering logic for `_d_newitemT`.
We were having trouble initialising the context pointer for nested structs.
The lowering has to perform 3 or 4 steps:

1. call `_d_newitemT`
1. initialise the `struct` (either with 0 or using the default initialiser)
1. (if the struct is nested) copy the context pointer
1. call the struct's ctor

Step #3 is difficult to perform in the frontend as it does not have any information about the stack.
The old lowering did not have this problem because it took place in the glue layer, where there is specific machinery for finding and setting the right context pointer.

Therefore, our solution was to split the lowering between semantic analysis and the glue layer.
Step 2 will be handled by the runtime hook itself, by calling [`emplaceInitializer`](https://github.com/dlang/dmd/blob/06295374255a31fa167de781bddcf70034e95e0e/druntime/src/core/internal/lifetime.d#L92).
Now only step 1 takes place in the frontend, while steps 3 and 4 are still performed by the glue layer.
Additionally, according to the feedback we received from [Johan](https://forum.dlang.org/post/wrtfsjheteinguxmfprv@forum.dlang.org), [Iain and Dennis](https://github.com/dlang/dmd/pull/14550#discussion_r1034661632), I have also kept the original expression from the frontend, instead of replacing it altogether with its lowering.
This adds at least 2 benefits:

- not having to handle the call to `_d_newitemT` separately during CTFE
- leveraging some optimisations in LDC and GDC that [Iain pointed out](https://forum.dlang.org/post/lncqlesvnjjtxxlydxbw@forum.dlang.org)

I created [this PR](https://github.com/dlang/dmd/pull/14664/) that encompasses all of the insights above.
This week I plan on getting the last failing test working and have it merged.
One of the failing tests was due to [this bug](https://issues.dlang.org/show_bug.cgi?id=23534) which has already been fixed. 
I will follow the same logic for `_d_newclass` and `_d_newarray*`.

I also fixed the final bugs with [`_d_arraycatnTX`](https://github.com/dlang/dmd/pull/14550).
However, moving the lowering to a new field in the `CatExp` class currently causes a bug in the compiler's backend that I am currently trying to fix.

Thanks,\
Teodor
