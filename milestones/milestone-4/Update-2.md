# [SAoC 2021] Replace druntime Hooks with Templates: Milestone 4, Week 2

Hi,

This week I implemented a template version of `_d_newThrowable` and I'm currently working on moving the lowering to it from e2ir.d to expressionsem.d.
This requires a call to `_d_newThrowable`, as well as setting `vthis` and `vthis2`, if the exception's class is nested, and then calling its ctor.
I'm still working on the latter steps of the lowering.

At the same time, I started looking at `_d_arraycatnTX`, starting from Dan Printzell's work:
- https://github.com/dlang/dmd/pull/10064
- https://github.com/dlang/druntime/pull/2648/

However, Dan's templated hook is actually just a wrapper that calls the old hook.
This makes both the hook itself and the lowering difficult to maintain and inefficient, because of the 2D-array that's created by the compiler before being linearised.
Thus, after I've decided to reimplement `_d_arraycatnTX` to use variadic templates instead of the 2D-array trick.
This will make the implementation much cleaner in both the compiler and in DRruntime.
I'm currently stuck on the `_d_arraycatnTXTrace` alias, however.
As it's [currently implemented](https://github.com/dlang/druntime/blob/759e60231a12482a1e1df5f891964e270dae0a1b/src/core/internal/array/concatenation.d#L55), the hook for `traceGC` doesn't seem to be instantiable with a variadic template `Tarr`.
I'm still exploring solutions for this problem, but I might have to implement the hook myself, without relying on `HookTraceImpl`.

Happy New Year!

Thanks,\
Teodor
