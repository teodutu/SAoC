# Replace `druntime` Hooks with Templates - Planning

The functions that will have to be replaced are listed here:
```
druntime $ egrep -o -R "_d_.*\(.*[^;]$" src/ | egrep "(Type|Class)Info" | egrep -v '`' | cut -d ':' -f 2 | cut -d '(' -f 1 | sort -u
_d_arrayappendcTX
_d_arraycatnTX
_d_arraycatT
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
_d_newarrayU
_d_newclass
_d_newitemiT
_d_newitemT
_d_newitemU
```

There are 23 non-template hooks.
They can be grouped according to functionality.
This makes it easier to convert them to templates.
Some can be converted to the same template, thus reducing the both codebase and the size of the resulting executables:

| Functionality                            | Hooks                                                                                                                        |
|:----------------------------------------:|:----------------------------------------------------------------------------------------------------------------------------:|
| Array concatenation (`a ~ b`)            | `_d_arraycatT`<br />`_d_arraycatnTX`                                                                                         |
| `struct` and `class` creation            | `_d_newitemiT`<br />`_d_newitemT`<br />`_d_newitemU`<br />`_d_newclass`                                                      |
| Array creation                      | `_d_newarrayiT`<br />`_d_newarraymiTX`<br />`_d_newarraymTX`<br />`_d_newarrayOpT`<br />`_d_newarrayT`<br />`_d_newarrayU`   |
| Array / associative array literal creation | `_d_arrayliteralTX`<br />`_d_assocarrayliteralTX`                                                                            |
| Array (re)allocation                     | `_d_arrayappendcTX`<br />`_d_arraysetcapacity`<br />`_d_arraysetlengthiT`<br />`_d_arraysetlengthT`<br />`_d_arrayshrinkfit` |
| Checks and casts                         | `_d_dynamic_cast`<br />`_d_interface_cast`<br />`_d_isbaseof`<br />`_d_isbaseof2`                                            |

[Vild](https://github.com/vild) worked on some hooks for GSOC 2019.
I have picked up some of his work last year.
This year I plan to finish his work:
- Vild converted [`_d_arraysetlengthT`](https://github.com/dlang/druntime/pull/2656) and [`_d_arrayappendcTX`](https://github.com/dlang/druntime/pull/2632) to templates, but he did so by creating template wrappers that call the old hooks.
I plan to reimplement those templates without calling the older hooks.
- Vild also made PRs to convert `_d_arraycatnTX` to a template:
    - DRuntime: https://github.com/dlang/druntime/pull/2648
    - DMD: https://github.com/dlang/dmd/pull/10064
I will draw inspiration from his work, but will take a different approach by converting `_d_arraycatnTX` to a variadic template, which will also cover the use cases of `_d_arraycatT`.

Therefore, considering my and Vild's previous work and the grouping above, my desired schedule looks as follows:
|          Milestone                   | Activity / Functions                                                                                                                                                              |      Details                                |
|:------------------------------------:|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|:------------------------------------------------------------------------------------------:|
| Milestone 1: 15.09.2022 - 15.10.2022 | `_d_arraycatT`<br />`_d_arraycatnTX`<br/>`_d_newitemiT`<br />`_d_newitemT`<br />`_d_newitemU`<br />`_d_newclass`                                                                  | Already started working on `_d_arraycatnTX`                                                |
| Milestone 2: 15.10.2022 - 15.11.2022 | `_d_newarrayiT`<br />`_d_newarraymiTX`<br />`_d_newarraymTX`<br />`_d_newarrayOpT`<br />`_d_newarrayT`<br />`_d_newarrayU`<br />`_d_arrayliteralTX`<br />`_d_assocarrayliteralTX` | |
| Milestone 3: 15.11.2022 - 15.12.2022 | `_d_arrayappendcTX`<br />`_d_arraysetcapacity`<br />`_d_arraysetlengthiT`<br />`_d_arraysetlengthT`<br />`_d_arrayshrinkfit`                                                      | `_d_arrayappendcTX` and `_d_arraysetlengthT` were partially converted to templates by Vild |
| Milestone 4: 15.12.2022 - 15.01.2023 | `_d_dynamic_cast`<br />`_d_interface_cast`<br />`_d_isbaseof`<br />`_d_isbaseof2` | |

I have purposely scheduled the "smaller" groups to the last 2 milestones to give me time to make up for any delays that may intervene throughout my schedule, either to University-related work, or to unforeseen events.
