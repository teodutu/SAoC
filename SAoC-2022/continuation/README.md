# `_d_araycatnTX`

> <https://github.com/dlang/dmd/pull/14550>

I finally finished the lowering after a rollercoaster of bugs.
Storing the lowering in `CatExp.lowering` turned out to be highly troublesome.

## `inline.d`

One of the first issues was related to inlining the call to the hook.
Previously there was no `visit()` method for `CatExp`.
`visit(BinExp)` was used instead.
We've added `visit(CatExp)` which inlines **either** the lowering, or the lhs and rhs of the `CatExp`.
Inlining both of them caused faulty Assembly code to be generated.
Too much space was allocated on the stack for one of the tests and one of the arguments of `_d_arraycatnTX()` was uninitialised and allocating memory for that length (a huge number) caused an `OutOfMemoryError`.

## Skipping `goto` Error for `_arrayliteral_on_stack`

[This PR](https://github.com/dlang/dmd/pull/14916) introduced [this test](https://github.com/dlang/dmd/blob/master/compiler/test/runnable/test23710.d), which was failing because the `auto ref` parameters of `_d_arraycatnTX()` save `lvalue`s to temporary variables that are declared on the spot.
Therefore, the `goto` statement was skipping over the declaration of that variable.
The fix was to ignore checking whether temporaries are by adding `STC.exptemp` to the `storage_class` of said temporaries.

## Skipping Codegen for `CatExp` when Compiling with `-betterC`

This one bug was the most difficult both to track down and to fix.
My troubles were introduced by [this test](https://github.com/dlang/dmd/blob/master/compiler/test/compilable/test23606.d).
`sc.flags == 0` when semantically analysing the `a ~ b`.
This made my code introduce the lowering despite it not being necessary, thus triggering a GC-related error (the GC is unavailable with `-betterC`).
I tried in vain to make `sc.flags & SCOPE.ctfe` non-zero.
The problem was that when analysing `foo()` for the first time, the `_scope` of the function is not the calling scope, but the one of its `TemplateDeclaration()`.
Being declared globally, it makes sense that `_scope.flags == 0`.
But since dynamic arrays are unavailable with `-betterC`, I just decided to skip the lowering altogether if this flag is enabled and this seemed to have worked.
