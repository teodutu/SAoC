# [SAoC 2023] Replace druntime Hooks with Templates: Milestone 2 Report

## Things Done in Milestone 2

### Store the Lowering of `CatElemAssignExp`s in a Separate Field

I started this milestone trying to fix [this bug](https://issues.dlang.org/show_bug.cgi?id=24159).
It was caused by not emitting an error at runtime rather than at compile time when compiling an array concatenation with `-betterC`.
The issue was caused by my previous lowering of `CatElemAssignExp`s to `_d_arrayappendcTX` by replacing it with a `CommaExp` in the the AST like so:

```d
arr ~= a + b;

// was lowered to:
__tmp = a + b; _d_arrayappendcTX(arr, 1), arr[$ - 1] = tmp;
```

The way to fix the bug was to move the lowering to a separate `lowering` field inside the original `CatElemAssignExp`.
This also made CTFE easier.
Replacing the original AST node and the fact that the hook was calling `libc` functions meant that I previously had to rebuild the original `CatElemAssignExp` during CTFE, which was both ugly and inefficient.
Now CTFE can simply handle the original `CatElemAssignExp` and not worry about the lowering.

I also had to pay attention to the inliner and inline the lowering instead of the concatenation operands to preserve `vthis` all the way to the hook.
Furthermore, despite trying to avoid lowering to the hook when the `CatAssignExp` is only used during CTFE, I was unable to find a way to figure out when this was the case.
I've had this problem in the past with [`_d_arraycatnTX`](https://github.com/dlang/dmd/pull/14550) and the solution was to move the check in the IR generator.
The reasoning is that if the `CatExp` does get to the IR generator, it means that it truly requires code generation (and is not just part of a `mixin` or something) and so, the compiler can now issue an error if the GC is disabled (as is the case with `-betterC`).
The same logic applies to `CatAssignExp`s.

Given all of these, I was able to [raise the PR](https://github.com/dlang/dmd/pull/15791) to fix the bug and have it merged eventually.

### Template `_d_newarraymTX`

I also took care of [converting `_d_newarray{mTX.miTX,OpT}` to templates](https://github.com/dlang/dmd/pull/15819), which was merged during the first week of milestone 3.
Similarly to [`_d_newarray{U,T,iT}`](https://github.com/dlang/dmd/pull/15299), `_d_newaraym*` are now implemented by a single hook: `_d_newarraymTX`.
Previously the two hooks were used to differentiate between default-initialised and zero-initialised types.
`_d_newarrayOpT` was the common implementation called by both `_d_newarraym{i,}TX` hooks.
`_d_newarrayOpT` received an alias template argument: either `_d_newarrayiT` or `_d_newarrayT` which it used to allocate the innermost 1-D array.
Now `_d_newarraymTX` can be made much simpler: it simply calls `_d_newarrayT`, which in turn uses DBI to figure out how to initialise the array.

### Weekly Updates

- [Week 1](https://forum.dlang.org/post/txzkqqliemmvjraaefef@forum.dlang.org)
- [Week 2](https://forum.dlang.org/post/rscxrjnneaujrqwqhaeb@forum.dlang.org)
- [Weeks 3 + 4](https://forum.dlang.org/post/gbpwiqfvxgwuomebdwhn@forum.dlang.org)

## Plans for Milestone 3

During the next milestone, I'll work on the following in parallel, juggling between them if I feel stuck or run out of ideas:

- using the template `_d_newarrayU` in `dup`
- `_d_arrayliteralTX`
- `_d_assocarrayliteralTX`.
