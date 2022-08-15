# [SAoC 2021] Replace druntime Hooks with Templates: Milestone 3, Week 3

Hi,

This week, I concluded that the best approach for `_d_arrayctor`, based on the performance calculated in [this post](https://forum.dlang.org/thread/hajlsppmugslhinluzos@forum.dlang.org) is to change its purity from strong to weak by adding a third, unused, pointer-type parameter.
This new signature was implemented in [this PR](https://github.com/dlang/druntime/pull/3587) and the PR for lowering to `_d_arrayctor` is waiting to be merged.
So it seems the `_d_arrayctor` saga is over for now.

Other than this, I started working on `_d_delstruct`, which I converted to a template in druntime [here](https://github.com/dlang/druntime/pull/3639).
I was also able to update its lowering in the compiler.
The only inconvenience now is that the call to `_d_delstruct` is also subject to semantic analysis, as opposed to the old hook, which was introduced in a later phase, by e2ir.d.
Because the new hook is analysed semantically, some compiler tests in `fail_compilation/` also output `@safe`-ty, purity and `nogc` errors related to `_d_delstruct`, and not just those coming from the dtor that's being called by `delete`.
I am currently working on a way to filter out the errors that mention `_d_delstruct`, as lowerings should be transparent to programmers.

Also this week I spent some time fixing the multiple calls to `_d_arrayappendcTX` that were created by the compiler for tests such as `(a ~= 1) ~= 2`.
The reason is that the lowering of `a ~= x` is actually this:
```d
__ctfe ? a ~= x : _d_arrayappendcTX(a, 1), a[$ - 1] = x, a;
```
Because `a` is used literally, as an expression, in case it's something more complex, such as, in this case, `a ~= 1`, the concatenation is made each time `a` appears in the second term of the conditional statement.
I am now trying to solve the error by saving the result of `a` in a temporary variable, in case it's not a regular array.

Thanks,\
Teodor
