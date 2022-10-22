# [SAoC 2022] Replace druntime Hooks with Templates: Milestone 1 Report

My project wants to replace dmd hooks to druntime functions such as `_d_arraycatnTX`, `_d_newclass` and `_d_newitemT` with templates.
This will make lowerings to these functions faster at runtime, as their templated versions will not require all the `TypeInfo`-related indirection used by the current hooks.
As a result, the maintainability of the compiler will be improved as lowerings will now take place in the semantic analysis phase, as opposed to the IR phase (where the hooks are currently present).
In addition, the compiler will also infer function attributes such as `pure` and `@nogc` when lowering to the new hooks.
The compiler will only instantiate those DRuntime functions that the app actually calls, thus reducing the size of D executables.
This will make deploying such apps on microcontrollers easier.

This is a continuation of my project from last year, during which I converted 10 hooks to templates:
```
_d_arrayctor
_d_arraysetctor
_d_arrayassign
_d_arrayassign_l
_d_arrayassign_r
_d_arraysetassign
_d_delstruct
_d_newThrowable
_d_arrayappendT
_d_arrayappendcTX
```

The project's goals are stated in [this issue](https://github.com/dlang/projects/issues/25) and it is tracked in [this project](https://github.com/orgs/dlang/projects/10/views/1).

## Week 1

I started the first week by diving into `_d_arraycatT` and `_d_arraycatnTX`.
`_d_arraycatT` was used for `a ~ b`, whereas `_d_arraycatnTX` is used for chained concatenations such as `a ~ b ~ c`.
I saw this as inefficient because:

1. it required 2 hooks for the same type of expression
1. `_d_arraycatnTX` handled concatenations by creating an array literal containing all the concatenated arrays and then passing this array to the hook like so: `tmp = [a, b, c], _d_arraycatnTX(tmp)`.
This was inefficient and made the lowering logic difficult and overly complicated.

My proposal was to unify `_d_arraycatT` and `_d_arraycatnTX` within one single template hook, `_d_arraycatnTX`, which takes variadic template arguments and is able to handle all kinds of concatenations without creating any extra arrays.

During my first week I implemented and tested the new `_d_arraycatnTX` hook and came up with a draft lowering.

## Week 2

I spent the second week fixing type mismatches when instantiating `_d_arraycatnTX`.
The issue was that I was trying to find a type `T` so that `_d_arraycatnTX!T(...)` would return a `T[]`.
This failed for strings, which had to be `immutable(char)[]`, so `T` became `immutable(char)`.
This made it impossible for me to construct the elements of the resulting string due to being immutable.
Casting away or forcing immutability when instantiating the hook also didn't work as it broke other code.

I also started working on `_d_newitem{T,iT,U}`.
They handle different cases of `new S()`, where `S` is a `struct`, namely whether to default-initialise the newly created object or not.
I planned to do something similar with them as I did with `_d_arraycat{T,nTX}`, which is to only create one template hook, `_d_newitemT` which should not perform any initialisation and have the compiler decide whether to add it or not.
This would reduce the number of hooks thus making them more maintainable.

## Week 3

With the help of my mentors, I reached a conclusion regarding the type mismatches from the previous week.
The solution was to simply instantiate `_d_arraycatnTX` with the type of the `CatExp` itself, after it had been semantically analysed and return that type as well.
While also being simpler and more straightforward, this took advantage of all casts previously insterted by the compiler and made sure the resulting expression was both modifiable and that the type of `_d_arraycatnTX` was consistent with that of the original expression.

After fixing all other bugs and making sure the rest of the tests pass, I got stuck on [`test19688.d`](https://github.com/dlang/dmd/blob/81f5c8b354aed2dc53a45e52498dc23f2f40fe88/compiler/test/runnable/test19688.d).
`__FUNCTION__` was incorrectly evaluated to an empty function.

## Week 4

It seemed like the previous bug was unrelated to my code and after asking Razvan if this was true, it turned out my gut feelign was correct.
The bug is tracked by [this issue](https://issues.dlang.org/show_bug.cgi?id=23408) and will be fixed by [this PR](https://github.com/dlang/dmd/pull/14549).

In the meantime, I have made a [PR](https://github.com/dlang/dmd/pull/14550) that converts `_d_arraycatnTX` to a template.
There are a few small issues regarding string concatenation.
At times such as [this one](https://github.com/dlang/dmd/blob/51c85d947692b0c389e4524d97ec168ffc0a7d48/compiler/test/unit/lexer/location_offset.d#L556), `.ptr` is used on a string type, which fails does not end in `\0`.
I am unsure now why this didn't cause errors in the past, and I am looking into how to find the root cause of this issue and fix it.

I also made further progress on the `_d_newitemT` front my making a draft lowering and I am now fixing it according to the failing tests.

## Milestone 2

During milestone 2 I plan to finish what's remaining from the first milestone:

- fixing the last bugs related to `_d_arraycatnTX`
- raising a PR for the template `_d_newitemT`

Furthermore, this milestone is dedicated to hooks that create new arrays or associative arrays.
I plan to start with `_d_newarray{iT,iTX,mTX,OpT,T,U}` and try to merge them similarly to `_d_newitemT`.

## Weekly Forum Posts

- [Week1](https://forum.dlang.org/post/vdviohlmzyokpktunslt@forum.dlang.org)
- [Week2](https://forum.dlang.org/post/jazxcjuiahfoqasmnaoj@forum.dlang.org)
- [Week3](https://forum.dlang.org/post/hfxvevognikgnabutvfm@forum.dlang.org)
- [Week4](https://forum.dlang.org/post/jbakvhuamylbrndfeako@forum.dlang.org)
