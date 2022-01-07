# [SAoC 2021] Replace druntime Hooks with Templates: Milestone 4, Week 3

Hi,

This week I finished converting `_d_newThrowable` to a template, as well as its lowering in the compiler.
The PRs for these are:
- https://github.com/dlang/druntime/pull/3661
- https://github.com/dlang/dmd/pull/13494

Also this week, I fixed the bug when compiling the new lowering for `a ~= b` to either `_d_arrayappendcTX` or `_d_arrayappendT`.
It turned out it was becaus of how `$` is evaluated by the compiler.
The initial lowering when `b` was a single element looked like this:
```d
__ctfe ? a ~= b : _d_arrayappendcTX(a, 1), a[$ - 1] = b, a;
```
When a chain of concatenations (like `(a ~= b) ~= c`) was lowered, in order for `(a ~= b)` to not be executed 3 times, I saved the result in a temporary variable, before concatenating `c` to it.
When this kind of expression was inlined, the `$` was mishandled by the compiler, which resulted in an incorrect `ArrayIndexError`.
To solve this issue, I used `a.length` instead of `$`.
Now the lowering looks like this:
```d
__ctfe ? a ~= b : _d_arrayappendcTX(a, 1), a[a.length - 1] = b, a;
// Note that the assignment above is actually a construction.
```
The PR that changes the lowering is: https://github.com/dlang/dmd/pull/13495.
A point of contention is the fact that I lower `a ~= b` to a `CondExp`.
As I explained [here](https://github.com/dlang/dmd/pull/13495#discussion_r780274768), I believe this is the more elegant approach, despite being more complex.

Another issue was that because the current template `_d_arrayappendcTX` uses a temporary implementation that calls the old hook using a mixin.
For this reason, the new hook is always impure and has to be declared `pure` so it can pass semantic analysis from `pure` contexts.
`_d_arrayappendT` also calls `_d_arrayappendcTX` and, hence, is declared `pure` as well.
However, `_d_arrayappendT` also calls `copyEmplace`, which can sometimes be im`pure`.
This causes an error when trying to instantiate `_d_arrayappendT`.
To bypass this, I created an im`pure` private function in DRuntime, which is called `_d_arrayappendT` after being made `pure` via a cast.
This way, `_d_arrayappendT` appears pure when it needs to be, but is also able to call an impure `copyEmplace` when it has to.

The implementation of this hack is here: https://github.com/dlang/druntime/pull/3662.\
Keep in mind that this is a hack for a stop-gap hook.
In the future, I intend to change the implementation of `_d_arrayappendcTX` so that it won't call the old hook anymore and thus allow attributes such as `pure`, `nothrow` etc to be inferred by the compiler.

Thanks,\
Teodor
