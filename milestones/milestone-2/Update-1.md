# [SAoC 2021] Replace druntime Hooks with Templates: Milestone 2, Week 1

The objectives of this milestone is to reach and implement a decision for the `_d_arrayctor` lowering and to translate the lowerings to `_d_arrayappendT` and `_d_arrayappendcTX` into templates.

This week, I picked up Dan's [work](https://github.com/dlang/dmd/pull/9982/) and started fixing the tests it fails.
The issue i'm working on now is related to CTFE.
Dan's original work avoided rewriting any expressions to the CTFE stack in dinterpret.d, but this lead to compilation errors.
I am currently working on adding CTFE support for lowerings to `_d_arrayappendcTX`, in a similar fashion to how Dan [handled `_d_arraysetlengthT`](https://github.com/dlang/dmd/blob/fe8cd9d09ef682a458ac6392042e660959e2778b/src/dmd/dinterpret.d#L4788-L4806).

In addition, my mentor Razvan has suggested a new approach for solving the issue caused by `_d_arrayctor`'s purity.
I mentioned this issue earlier in [this post](https://forum.dlang.org/post/simesvkancmscrtsciwq@forum.dlang.org).
This possible fix is to change `_d_arrayctor`'s definition so as to lower expressions such as:
```d
struct S {};
S[2] a;
immutable S[2] b = a;
```
to
```d
immutable S[2] b = _d_arrayctor!(immutable(S[2]))(a)
```
instead of
```d
immutable S[2] b;
_d_arrayctor(b, a);
```

This solution would keep `_d_arrayctor` pure while also making use of the value it returns.
There are, however, still some points to talk clarify about this change.
For example, one is with regards to the length and overlap [checks](https://github.com/dlang/druntime/blob/6a3cf6608b7ec7e238c2484f5ef858c8afa725f3/src/core/internal/array/construction.d#L35-L49) that the current implementation is performing.
We're still unsure how to proceed about those.

Next week, my mentors and I will explore this new approach to `_d_arrayctor` further and I'll proceed to implement it once all questions have been answered.
In addition, I will try to finish adding CTFE support for `_d_arrayappendcTX` and then move on to fixing the other failing tests.
