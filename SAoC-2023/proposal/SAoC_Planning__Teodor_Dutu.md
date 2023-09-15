# Replace `druntime` Hooks with Templates - Planning

The functions that will have to be replaced are listed here:

```text
_d_arrayappendcTX
_d_arrayliteralTX
_d_arraysetcapacity
_d_arraysetlengthiT
_d_arraysetlengthT
_d_arrayshrinkfit
_d_assocarrayliteralTX
_d_dynamic_cast
_d_interface_cast
_d_isbaseof
_d_isbaseof2
_d_newarrayiT
_d_newarraymiTX
_d_newarraymTX
_d_newarrayOpT
_d_newarrayT
_d_newarrayiT
_d_newarrayU
```

There are 18 non-template hooks.
They can be grouped according to functionality.
This makes it easier to convert them to templates.
Some can be converted to the same template, thus reducing the both codebase and the size of the resulting executables:

| Functionality                              | Hooks                                                                                                                        |
|:------------------------------------------:|:----------------------------------------------------------------------------------------------------------------------------:|
| 1-dimensional array creation               | `_d_newarrayiT`<br />`_d_newarrayT`<br />`_d_newarrayU`                                                                     |
| Multi-dimensional array creation           | `_d_newarraymiTX`<br />`_d_newarraymTX`<br />`_d_newarrayOpT`                                                                |
| Array / associative array literal creation | `_d_arrayliteralTX`<br />`_d_assocarrayliteralTX`                                                                            |
| Array (re)allocation                       | `_d_arrayappendcTX`<br />`_d_arraysetcapacity`<br />`_d_arraysetlengthiT`<br />`_d_arraysetlengthT`<br />`_d_arrayshrinkfit` |
| Checks and casts                           | `_d_dynamic_cast`<br />`_d_interface_cast`<br />`_d_isbaseof`<br />`_d_isbaseof2`                                            |

It is worth noting that `_d_newarrayOpT` already is a template called by `_d_newarraymiTX` and `_d_newarraymTX` (depending on whether the type of the multi-dimensional array has an initialiser or is zero-initialised).
In turn, `_d_newarrayOpT` calls `_d_newarrayT` or `_d_newarrayiT`, according to the type's initialiser.
Therefore, converting multi-dimensional array hooks to templates will only require changing the wrappers (`_d_newarraymiTX` and `_d_newarraymTX`) and lowering logic.

[Vild](https://github.com/vild) worked on some of these hooks for GSOC 2019.
Specifically, he converted [`_d_arraysetlengthT`](https://github.com/dlang/druntime/pull/2656) and [`_d_arrayappendcTX`](https://github.com/dlang/druntime/pull/2632) to templates, but he did so by creating template wrappers that call the old hooks.
I plan to reimplement those templates without calling the older hooks and remove the latter.

Razvan [tried to convert `_d_arrayshrinkfit` to a template](https://github.com/dlang/dmd/pull/15289), but ran into some issues regarding importing `rt/lifetime.d` into `core/internal/lifetime.d`.
This meant that some utility functions from `rt/lifetime.d`, such as `__setArrayAllocLength`, or `__arrayAlloc` will have to be templated alongside some of the hooks that use them.

Therefore, considering my and Vild's previous work and the grouping above, my desired schedule looks as follows:
|          Milestone                   | Activity / Functions                                                                                                                                                              |      Details                                |
|:------------------------------------:|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|:------------------------------------------------------------------------------------------:|
| Milestone 1: 15.09.2023 - 15.10.2023 | `_d_newarrayU`<br />`_d_newarrayT`<br />`_d_newarrayiT`<br />`_d_newarraymTX`<br />`_d_newarraymiTX`<br /> |
| Milestone 2: 15.10.2023 - 15.11.2023 | `_d_arrayliteralTX`<br />`_d_assocarrayliteralTX` | |
| Milestone 3: 15.11.2023 - 15.12.2023 | `_d_arrayappendcTX`<br />`_d_arraysetcapacity`<br />`_d_arraysetlengthiT`<br />`_d_arraysetlengthT`<br />`_d_arrayshrinkfit`                                                      | `_d_arrayappendcTX` and `_d_arraysetlengthT` were partially converted to templates by Vild |
| Milestone 4: 15.12.2023 - 15.01.2023 | `_d_dynamic_cast`<br />`_d_interface_cast`<br />`_d_isbaseof`<br />`_d_isbaseof2` | |
