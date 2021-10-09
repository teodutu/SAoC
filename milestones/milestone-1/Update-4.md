# [SAoC 2021] Replace druntime Hooks with Templates: Milestone 1, Week 4

Hi,

The PR that replaces the `_d_arrayctor` and `_d_arraysetctor` hooks is blocked
by a bug in the compiler's backend, which was posted as an issue [here](https://issues.dlang.org/show_bug.cgi?id=22372).

In the meantime, I improved the error messages for the `_d_arrayctor` and `_d_arraysetctor` templates in druntime.
For an [older PR](https://github.com/dlang/druntime/pull/3582), I had to resort to using some rather austere error messages because of the requirement that `_d_arrayctor` be `@nogc`.

However, following my mentor Edi's [advice](https://github.com/dlang/druntime/pull/3582#discussion_r725480441), I implemented `@nogc` versions of the array utils functions inside `core/internal/util/array.d` and updated `_d_arrayctor` to use `enforceRawArraysConformableNogc`, in [this PR](https://github.com/dlang/druntime/pull/3583).

TODO: another hook... anything...

Thanks,\
Teodor
