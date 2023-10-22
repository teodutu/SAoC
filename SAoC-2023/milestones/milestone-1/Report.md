# [SAoC 2023] Replace druntime Hooks with Templates: Milestone 1 Report

## Things Done in Milestone 1

At the beginning of this milestone I worked [on `_d_newarrayT`](https://github.com/dlang/dmd/pull/15299).
The new template was intended to replace all `_d_newarray{T,iT,U}`.
Previously, `_d_newarrayT` was used for zero-initalised types and `_d_newarrayiT` for those with default initialisers.
`_d_newarrayU` was the common implementation that allocated the array and was used by both of the aforementioned hooks.
Using DBI, the template `_d_newarrayT` could inspect the types of the array elements and thus could decide if they needed initialisers or not by itself.
This removed the need to separate between `_d_newarrayT` and `_d_newarrayiT`.

In addition, these hooks used various auxiliary functions that could not be imported from `rt/lifetime.d` because of how DRuntime is compiled.
Therefore, I added `__arrayClearPad`, `__arrayAlloc`, `__arrayStart` and `__setArrayAllocLength` as templates to `core/internal/array/utils.d`.
I couldn't remove them from `rt/lifetime.d` as they're still used by other hooks that haven't been converted to templates yet.

Furthermore, `dup` uses `_d_newarrayU` and it would have been meaningless to make it call `_d_newarrayT` and initialise the new array before overwriting those initialised elements with the contents of the duplicated array.
So `_d_newarrayU` remained.
Unfortunately, when trying to update `dup` to use `_d_newarrayU` something broke inside the GC and `collect` started throwing some errors.
Therefore I decided to simplify the [PR](https://github.com/dlang/dmd/pull/15299) for converting `_d_newarray{T,iT,U}` to templates by just handling direct lowerings to `_d_newarray{T,iT}`.
Replacing the call from `dup` to `_d_newarrayU` will be handled in a future PR.

[This bug](https://issues.dlang.org/show_bug.cgi?id=24159) showed up next and it was caused by my [previous work on template `_d_arrayappend{T,cTX}`](https://github.com/dlang/dmd/pull/13495).
I worked on these hooks some time ago and handled them rather poorly by replaced the `CatAssignExp` with the call to the hook in the AST itself.
This created issues during CTFE as hooks are generally not interpretable because they often call `libc` functions.

The "correct" way to handle lowerings is to introduce a `lowering` field inside the original expression in `expression.d` and use this field to store the lowered expression.
This way CTFE can evaluate the original expression and ignore the lowering.
At the same time, the glue layer can generate its IR from the lowering and not the original expression.
Now I use this approach for all newly converted hooks and I intend to come back to my initial work from 1-2 years ago and apply this new methodology to those hooks as well.

Coming back to the bug, the lowered expression of `arr ~= elem` remains the same `_d_arrayappendcTX(arr, 1), arr[$ - 1] = elem`.
However, now this is causing a backend error when `arr` is a function call, say `foo()`.
To avoid calling `foo()` twice, I save its return value to a temporary variable and use that instead in the `CommaExp` as follows:

```d
foo() ~= elem;

// is lowered to:
_tmp = foo(), _d_arrayappendcTX(_tmp, 1), _tmp[$ - 1] = elem
```

The issue is that the backend cannot find the `_tmp` symbol.
Most likely the lowering isn't properly analysed in the frontend.
Previously, because the original `CatAssignExp` node was replaced with a `CallExp` essentially and this made the new node pass through all the checks and analyses that it was supposed to.
Now however, I need to manually perform those passes on the `lowering` field and first I need to find the right ones.

In parallel, I started working on using the new template `_d_newarrayU` from `dup` and on `_d_newarraym{T,iT}` (for creating multi-dimensional arrays).
Both rely on [the `_d_newarrayT` PR](https://github.com/dlang/dmd/pull/15299) despite it not being merged yet.
The reason for it not being merged is that MacOS tests fail [when installing prerequisites](https://github.com/dlang/dmd/actions/runs/6596639161/job/17922891698?pr=15299) and that the monthly free compute limit for GitHub actions has been exceeded.
Regarding `_d_newarraym{T,iT}`, they both already call a template function `_d_newarrayOpT`.
In turn, this template calls either `_d_newarrayT` or `_d_newarrayiT` to initialise the arrays recursively.
I've created an initial implementation of a template `_d_newarraymT`, but it still returns `void[]`.
I'd like to have it return a typed array such as `int[][][]` or more generically `T[][]`.

## Plans for Milestone 2

This milestone I have started out on quite a few correlated paths (`dup`, `_d_newarraym{T,iT}` and [the bug regarding `_d_arrayappend{T,cTX}`](https://issues.dlang.org/show_bug.cgi?id=24159)).
I first intend to wrap these up in the following weeks before moving on to other hooks (`_d_arrayliteralTX` and `_d_assocarrayliteralTX`).
