# [SAoC 2021] Replace druntime Hooks with Templates: Milestone 1, Week 4

Hi,

The PR that replaces the `_d_arrayctor` and `_d_arraysetctor` hooks was blocked by a bug in the compiler's backend, which was posted as an issue [here](https://issues.dlang.org/show_bug.cgi?id=22372).
The issue was eventually fixed, which has allowed the aforementioned PR to make some progress.

However, another bug was revealed, whereby the `_d_arrayctor` [template function](https://github.com/dlang/druntime/blob/34b31e2b7ac6198255bca8d6d318a4f481b05604/src/core/internal/array/construction.d#L24) turned out to be incorrect.
When instantiating the function with `const` or `immutable` array type as argument, the compiler issued warnings when its return value was ignored, because `_d_arrayctor` is also a strongly pure function.
This means that the call to it seemingly has no effect.
Furthermore, this false impression might make the compiler remove the call to it altogether.

A temporary and hackish solution was to add a pointer type parameter to `_d_arrayctor`, implemented by [this PR](https://github.com/dlang/druntime/pull/3587).
This solution turns `_d_arrayctor` from  being **strongly pure** to being **weakly pure**.
As a result, its call cannot be optimised out by the compiler.

We're still looking for a more elegant solution to the issue above, as discussed in [this thread](https://forum.dlang.org/thread/simesvkancmscrtsciwq@forum.dlang.org).
One alternative is to change `_d_arrayctor`'s signature so that its 2 parameters are of different types - say `T1[]` and `T2[]` - so that `Unqual!T1 == Unqual!T2`.
I am currently working on this change.

In the meantime, I improved the error messages for the `_d_arrayctor` and `_d_arraysetctor` templates in druntime.
For an [older PR](https://github.com/dlang/druntime/pull/3582), I had to resort to using some rather austere error messages because of the requirement that `_d_arrayctor` be `@nogc`.

However, following my mentor Edi's [advice](https://github.com/dlang/druntime/pull/3582#discussion_r725480441), I implemented `@nogc` versions of the array utils functions inside `core/internal/util/array.d` and updated `_d_arrayctor` to use `enforceRawArraysConformableNogc`, in [this PR](https://github.com/dlang/druntime/pull/3583).

Thanks,\
Teodor
