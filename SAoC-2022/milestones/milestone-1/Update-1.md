# [SAoC 2022] Replace druntime Hooks with Templates: Milestone 1, Week 1

Hi,

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

I am currently working on translating `_d_arraycatT` and `_d_arraycatnTX` to templates.
Currently `_d_arraycatT` was used for `a ~ b`, whereas `_d_arraycatnTX` is used for chained concatenations such as `a ~ b ~ c`.
The latter lowering allocates a temporary array on the stack that stores `a`, `b` and `c`: `[a, b, c]` then passes this array to the hook as an argument.
This is inefficient and makes for an inelegant lowering.
I intend to replace both `_d_arraycatT` and `_d_arraycatnTX` with a new `_d_arraycatnTX` that takes arguments as a variadic template.
Thus, it should cover all use cases without allocating unnecessary arrays.

So far, I have implemented and tested the new `_d_arraycatnTX` hook.
I have also come up with a draft lowering that manages to compile druntime and phobos.
This was not easy, as they make extensive use of array concatenations, especially for strings.
Now I am working on fixing the lowering and the hook to make the tests in dmd pass.

Thanks,\
Teodor
