# [SAoC 2021] Replace druntime Hooks with Templates: Milestone 4, Week 4

Hi,

This week, following the discussions on the [PR](https://github.com/dlang/dmd/pull/13495) for lowering `_d_arrayappend{cTX,T}` to templates, my mentors and I decided we should probably change the implementation of the template hooks themselves.

Because of the fact that `_d_arrayappendT` only appends 2 arrays, if the rhs is a single element, the lowering of `lhs ~= rhs` currently has to look like this:
```d
_d_arrayappendcTX(lhs, 1), a[lhs.length - 1] = rhs, lhs;
```
This makes it impossible for CTFE to convert the lowering back to `lhs ~= rhs`.
Therefore, the complete lowering is:
```d
__ctfe ? lhs ~= rhs : _d_arrayappendcTX(lhs, 1), a[lhs.length - 1] = rhs, lhs;
```
This lowering is problematic because the IR generator has to discard the `CondExp` and its `true` branch manually.

We asked [this](https://forum.dlang.org/post/dcwxswfamadkdnflkaap@forum.dlang.org) question on the forum about the issue above.
In it, we propose to extend `_d_arrayappendT` to be able to concatenate both arrays and single elements to the lhs array, so that semantic3 can simply lower the `lhs ~= rhs` to `_d_arrayappendT` every time.

Additionally, I started working on the `_d_newitem{U,iT,T}` hooks.
It is likely we won't be able to remove the usage of `TypeInfo` from `_d_newitemU`.
The old hook writes the `TypeInfo` object at the end of the newly created `struct` object.
This is most likely in order to help the GC.
We asked yet another [question](https://forum.dlang.org/post/vmysmecdycsmuiuuqifn@forum.dlang.org) on the forum about this `TypeInfo` object and whether its removal would cause trouble or not.

Thanks,\
Teodor
