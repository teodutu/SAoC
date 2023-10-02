# [SAoC 2023] Replace DRuntime Hooks with Templates Weekly Update #2

Hi,

Last week I started working on converting `_d_newarraym{i,}T` to templates starting from the `_d_newarray{i,}T` implementations from [my previous PR](https://github.com/dlang/dmd/pull/15299/).
So far I have implemented the new hooks and updated the lowering but ran into some test failures in the compiler.

Before fixing them, [this bug](https://issues.dlang.org/show_bug.cgi?id=24159) showed up and it was caused by my [lacklustre lowering to `_d_arrayappend{T,cTX}`](https://github.com/dlang/dmd/pull/13495).
I worked on these hooks some time ago and replaced the `~=` with the call to the hook in the AST itself.
This created issues during CTFE as hooks are generally not interpretable because they often call `libc` functions.

The solution that I'm implementing now is to store the lowering of `~=` in a `lowering` field inside `CatAssignExp`.
Then CTFE can evaluate the original expression and ignore the lowering and the glue layer can generate its IR from the lowering and not the `CatAssignExp`.
The lowered expression of `arr ~= elem` remains the same `_d_arrayappendcTX(arr, 1), arr[$ - 1] = elem`.

However, now this is causing a backend error when `arr` is a function call, say `foo()`.
To avoid calling `foo()` twice, I save its return value to a temporary variable and use that instead in the `CommaExp` as follows:

```d
foo() ~= elem;

// is lowered to:

_tmp = foo(), _d_arrayappendcTX(_tmp, 1), _tmp[$ - 1] = elem
```

Somehow the backend cannot find the `_tmp` symbol.
I am still investigating this and how moving the lowering to another expression causes it.

Thanks,\
Teodor
