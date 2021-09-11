# Replace `druntime` Hooks with Templates

## Current Status
To perform runtime operations, such as calls to the GC and vector operations,
the D compiler rewrites user code to call specific functions from the runtime
library. These runtime functions are called "runtime hooks". These hooks are
currently implemented as simple functions. This introduces the following
disadvantages:
- The entire `druntime` library needs to be linked against any executable
written in D. Thus, when linking the library statically, all `druntime`
functions are present in the resulting executables. This makes it difficult to
deploy such apps on low memory platforms, such as microcontrollers, because the
size of `libdruntime.a` is approximately 13 MB, on my own 64-bit Linux system.
- The current mechanism is using the `TypeInfo` class hierarchy to pass runtime
information to `druntime` functions. The `TypeInfo` hierarchy is implemented
using classes, therefore it has the following shortcomings:
    1. slow dynamic dispatch when choosing what override to call at runtime;
    2. the GC is required to manage the `TypeInfo` classes.
- Currently, the runtime hooks are inserted at the IR level of the compiler
passes, which is a difficult and hard to understand component of the compiler.

## Goals
The aim of this project is to replace the existing runtime hooks with templates
in order to address the above disadvantages, that are brought by using hooks. As
such, this project will add the following improvements to the D ecosystem:
- The compiler will only instantiate those `druntime` functions that the app
actually calls, thus reducing the size of D executables. This will make
deploying such apps on microcontrollers easier.
- By using templates, the `TypeInfo` class becomes unnecessary, as all type
information is available at compile time. Thus, `TypeInfo` will be removed
completely from both `DMD` and `druntime` as its usage will become obsolete. As
a result, all its dependencies mentioned above will no longer be required. For
this reason, instead of downcasting `TypeInfo` to the actual class at runtime,
`DMD` could perform these operations at compile time, together with all the
necessary runtime checks.
- Maintainability of both code bases will be improved as using templates will
result in less code duplication in `druntime` and a cleaner compiler front-end
implementation (because types are deduced during semantic analysis and not
during IR).

All these changes are completely in line with and will enhance D's features,
such as design by introspection, metaprogramming, and CTFE. In addition, this
project will only bring changes inside `DMD` and `druntime`, without changing
the language as a whole or its grammar. For this reason, after this project is
completed, any previous D code will continue to work, even with some performance
improvements and without needing any code changes.

## Method
The desired approach follows in the steps outlined by Vild in
[his work for GSoC 2019](https://raw.githubusercontent.com/Vild/GSOC2019/master/Proposal/Proposal.pdf#section.3).
Specifically, this project will further replace the existing runtime hooks with
templates, function by function. For each function, the conversion would require
two steps, that need to be performed in order:
1. The first will involve adding `druntime` support for the reworked template
function, that no longer uses `TypeInfo` and write `unittests` for the new use
cases. A working example of the required changes can be seen
[in this PR](https://github.com/dlang/druntime/pull/2656), belonging to Vild.
2. The next step will be to change how the compiler handles the `druntime` call
by using the previously added template function, instead of indirecting it via
a hook. Once again, an example of how this would look like, for the
[`_d_arraysetlengthT` function](https://github.com/dlang/dmd/pull/10106/) has
been implemented by Vild.

After the PRs associated with the above steps are merged, a new PR will be
required in order to clean up the old `druntime` implementation of each function
that would still be using `TypeInfo`.

Special attention will need to be paid to the `pure @safe nothrow` qualifiers.
While `@safe` and `nothrow` can be enforced by using `@trusted` and by catching
any thrown errors "manually" and then using `assert(0)`, `pure` will probably
require the use of the specific `pure` variants of the functions exposed by
`druntime`, such as the ones regarding
[memory allocation](https://github.com/dlang/druntime/blob/78367e154c9122038f6e85424cb8088a2b610307/src/core/memory.d#L1007).

## Planning
The functions that will have to be replaced are listed here:
```bash
$ egrep -o -R "_d_.*\(.*[^;]$" | grep "TypeInfo" | cut -d ':' -f 2 | egrep -o "_d_[^(]*" | sort -u
_d_arrayappendcTX
_d_arrayappendT
_d_arrayassign
_d_arrayassign_l
_d_arrayassign_r
_d_arraycatnTX
_d_arraycatT
_d_arrayctor
_d_arrayliteralTX
_d_arraysetassign
_d_arraysetcapacity
_d_arraysetctor
_d_arraysetlengthiT
_d_arraysetlengthT
_d_arrayshrinkfit
_d_assocarrayliteralTX
_d_delarray_t
_d_delstruct
_d_isbaseof
_d_newarrayiT
_d_newarraymiTX
_d_newarraymTX
_d_newarrayOpT
_d_newarrayT
_d_newarrayU
_d_newitemiT
_d_newitemT
_d_newitemU
_d_newThrowable
```

For the duration of SAoC, a subset of the above functions will be reworked to
use templates instead of hooks, according to the following schedule:
|          Week           |                                 Activity / Functions                               |      Details     |
|:-----------------------:|:----------------------------------------------------------------------------------:|:----------------:|
| 15.09.2021 - 22.09.2021 | `_d_arrayappendcTX`                                                                |
| 22.09.2021 - 29.09.2021 | `_d_arrayappendT`                                                                  |
| 29.09.2021 - 06.10.2021 | `_d_arrayassign`<br />`_d_arrayassign_l`                                           |
| 06.10.2021 - 13.10.2021 | `_d_arrayassign_r`<br />`_d_arraycatnTX`<br />**Performance Evaluation**           |
| 13.10.2021 - 20.10.2021 | `_d_arraycatT`<br />`_d_arrayctor`                                                 | First Milestone  |
| 20.10.2021 - 27.10.2021 | `_d_arraysetctor`<br />`_d_arraysetassign`                                         |
| 27.10.2021 - 03.11.2021 | `_d_arraysetcapacity`<br />`_d_arraysetlengthiT`                                   |
| 03.11.2021 - 10.11.2021 | `_d_arraysetlengthT`<br />`_d_assocarrayliteralTX`<br />**Performance Evaluation** |
| 10.11.2021 - 17.11.2021 | `_d_arrayliteralTX`<br />`_d_delarray_t`                                           | Second Milestone |
| 17.11.2021 - 24.11.2021 | `_d_delstruct`<br />`_d_arrayshrinkfit`                                            |
| 24.11.2021 - 01.12.2021 | `_d_isbaseof`<br />`_d_newarrayiT`                                                 |
| 01.12.2021 - 08.12.2021 | `_d_newarraymiTX`<br />`_d_newarraymTX`                                            |
| 08.12.2021 - 15.12.2021 | `_d_newarrayOpT`<br />`_d_newarrayT`                                               | Third Milestone  |
| 15.12.2021 - 22.12.2021 | `_d_newarrayU`<br />`_d_newitemiT`                                                 |
| 22.12.2021 - 29.12.2021 | `_d_newitemT`<br />`_d_newitemU`                                                   |
| 29.12.2021 - 05.01.2022 | `_d_newThrowable`<br />**Final Evaluation**                                        |
| 05.01.2022 - 15.01.2022 | Cleanup PR                                                                         | Final Milestone  |


