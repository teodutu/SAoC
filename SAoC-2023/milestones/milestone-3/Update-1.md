# [SAoC 2023] Replace DRuntime Hooks with Templates Weekly Update #9

Hi,

This week I finished working on converting `_d_newarray{mTX,miTX,OpT}` to a single template and got the [PR](https://github.com/dlang/dmd/pull/15819) merged.
They allocate and initialise multi-dimensional arrays.
Similarly to [`_d_newarray{U,T,iT}`](https://github.com/dlang/dmd/pull/15299), `_d_newaraym*` are now implemented by a single hook: `_d_newarraymTX`.
Previously the two hooks were used to differentiate between default-initialised and zero-initialised types.
`_d_newarrayOpT` was the common implementation called by both `_d_newarraym{i,}TX` hooks.
`_d_newarrayOpT` received an alias template argument: either `_d_newarrayiT` or `_d_newarrayT` which it used to allocate the innermost 1-D array.

Now `_d_newarraymTX` can be made much simpler: it simply calls `_d_newarrayT`, which in turn uses DBI to figure out how to initialise the array.
In the following weeks I'll start working on `_d_arrayliteralTX` and `_d_assocarrayliteralTX`.

Thanks,\
Teodor
