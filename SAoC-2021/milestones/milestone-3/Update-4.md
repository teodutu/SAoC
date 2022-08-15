# [SAoC 2021] Replace druntime Hooks with Templates: Milestone 3, Week 4

Hi,

I spent most of this week working on replacing `_d_delstruct` with a template.
I first did it through [this PR](https://github.com/dlang/druntime/pull/3639) for druntime, followed by [this one](https://github.com/dlang/dmd/pull/13398) to change the lowering.
Razvan [pointed out to me](https://github.com/dlang/dmd/pull/13398#discussion_r764625641) that I had made a mistake by not considering the case when the `-gc_profile` parameter was given.
Hence, I wrapped `_d_delstruct` in a template and added a new hook called `_d_delstructTrace` as an alias to the already implemented `_d_delstruct` [here](https://github.com/dlang/druntime/pull/3644).

In the meantime, I resumed the work on the `_d_arrayappendcTX` and `_d_arrayappendT` hooks.
I fixed my error from the previous week and now tests are running fine in dmd and druntime.
There are still, however, a few errors when testing phobos.
They are caused by the fact that `_d_arrayappendT` is declared `pure` (and needs to be so in order to be lowererd to from pure contexts), but calls `copyEmplace`, which, in those cases, is not pure because of the postblit it calls.
I am still investigating and trying to narrow down the cases when impure postblits trigger this error, yet don't trigger errors when the origninal expression passes through semantic analysis.

Furthermore, when fixing the bug from last week, I had to determine which expressions `a` can be used unaltered in the lowering below, and which have to be saved in a temporary variable first.
```d
__ctfe ? a ~= x : _d_arrayappendcTX(a, 1), a[$ - 1] = x, a;
```
For now, the decision is oversimplified as it only checks for a few expression types such as variables or index expressions.
Index expressions cannot be saved to temporaries because of [this bug](https://issues.dlang.org/show_bug.cgi?id=9023).
Saving `s["a"]` in a variable would trigger a CTFE error, because `s` is uninintialised.
However, if `a` from the lowering above is `foo()[i]`, then `a` is an index expression, but the function call can also have side effects, which makes it necessary to use a temporary variable here, as well.
And what if `foo()` itself returns an uninitialised associative array?
I'll have to give this logic some more thought over the course of next week.

Thanks,\
Teodor
