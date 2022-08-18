# Replace DRuntime hooks with templates

## Current Status

To perform runtime operations, such as calls to the GC and vector operations, the D compiler rewrites user code to call specific functions from the runtime library.
These runtime functions are called "runtime hooks".
The hooks are currently implemented as simple functions.
This has the following disadvantages:
- The entire DRuntime library needs to be linked against any executable written in D.
Thus, when linking the library statically, all DRuntime functions are present in the resulting executables.
This makes it difficult to deploy such apps on low memory platforms, such as microcontrollers, because the size of `libdruntime.a` is approximately 16 MB on a 64 bit Linux system.
- The current mechanism is using the `TypeInfo` class hierarchy to pass runtime information to DRuntime functions.
The `TypeInfo` hierarchy is implemented using classes, therefore it has the following shortcomings:
    - slow dynamic dispatch when choosing what override to call at runtime;
    - the GC is required to manage the `TypeInfo` classes.
- Currently, runtime hooks are inserted at the IR level of the compiler passes, which is a difficult and hard to understand component of the compiler.
- In addition, no function attributes can be inferred for hooks and array arguments are `void[]` or `byte[]` instead of their actual types, which is unsafe.

## Goals

The aim of this project is to replace the existing runtime hooks with templates in order to address the above disadvantages, that are brought by using hooks.
As such, this project will add the following improvements to the D ecosystem:
- The compiler will only instantiate those DRuntime functions that the app actually calls, thus reducing the size of D executables.
This will make deploying such apps on microcontrollers easier.
- By using templates, runtime hooks will no longer need to use `TypeInfo`.
Instead of obtaining type information at runtime, hooks will get this information at compile-time, thus improving performance at the cost of slightly longer compilation times.
- In addition to extracting type information such as object sizes and unqualified types, the compiler will also infer function attributes such as `pure` and `@nogc` when lowering to the new hooks.
- Maintainability of both code bases will be improved as using templates will result in less code duplication in DRuntime and a cleaner compiler front-end implementation (because types are deduced during semantic analysis and not during IR).

All these changes are completely in line with and will enhance D's features, such as design by introspection, metaprogramming, and CTFE.
In addition, this project will only bring changes inside DMD and DRuntime, without changing the language as a whole or its grammar.
For this reason, after this project is completed, any previous D code will continue to work, even with some performance improvements and without needing any code changes.

## Method

The desired approach follows the steps outlined by Vild in [his work for GSoC 2019](https://raw.githubusercontent.com/Vild/GSOC2019/master/Proposal/Proposal.pdf#section.3)
Specifically, this project will further replace the existing runtime hooks with templates, function by function.
After the merge of dmd and DRuntime, only one PR will be required to translate one (or more, if similar) hook(s) to templates.
While previously this process required 3 PRs (one that added the new template hooks to DRuntime, another that changed their lowerings in dmd and a third one that removed the old hooks), now only one is needed, as shown in [this PR](https://github.com/dlang/dmd/pull/14310) that changed 3 hooks to templates: `_d_arrayassign`, `_d_arrayassign_l` and `_d_arrayassign_r`.

Many runtime hooks will have to be `@trusted`.
For example, those that construct or assign objects can be instantiated with `struct` types that define postblits.
In this case, those hooks need to call `memcpy`, which is `@system`.
In order to allow hooks to be called from `pure` contexts, any memory allocation they perform will use [the designated pure wrappers](https://github.com/dlang/dmd/blob/51b75544da5f227607cb2567a5063ebd540e49e0/druntime/src/core/memory.d#L1028).

## Previous Work

I have already started working on this project last year, [also as part of SAoC](https://forum.dlang.org/thread/ofpvwqnesyxjidcioqme@forum.dlang.org).
Up to now, I have converted 9 runtime hooks to templates:
```
_d_arrayctor
_d_arraysetctor
_d_arrayassign
_d_arrayassign_l
_d_arrayassign_r
_d_delstruct
_d_newThrowable
_d_arrayappendT
_d_arrayappendcTX
```

Using the lessons learned so far, I intend to continue my work during SAoC 2022 and beyond.
There are 25 runtime hooks that are not yet converted to templates:
```
_d_allocclass
_d_arrayappendcTX
_d_dynamic_cast
_d_interface_cast
_d_isbaseof
_d_isbaseof2
_d_newarrayiT
_d_arraycatnTX
_d_newarraymiTX
_d_arraycatT
_d_newarraymTX
_d_arrayliteralTX
_d_newarrayOpT
_d_arraysetassign
_d_newarrayT
_d_arraysetcapacity
_d_newarrayU
_d_newclass
_d_arraysetlengthiT
_d_newitemiT
_d_newitemT
_d_arrayshrinkfit
_d_newitemU
_d_assocarrayliteralTX
_d_delarray_t
```

For the duration of SAoC, I will rework a subset of the above functions and their lowerings to use templates.
