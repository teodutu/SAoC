# [SAoC 2021] Replace druntime Hooks with Templates: Milestone 1, Week 3

Hi,

The changes to DMD from the [previous week](https://github.com/dlang/dmd/pull/13116) made some druntime unittests fail.
Specifically, some of those in `core.lifetime`, because the existing `_d_arrayctor` implementation
could not be `@nogc`.

I spent this week fixing the above issue by removing the call to `enforceRawArraysConformable` from `_d_arrayctor`, and performing its checks "manually".
`enforceRawArraysConformable` prevented `_d_arrayctor` from becoming `@nogc` because it created its error messages by string concatenation, which is not `@nogc`.
The PR that _should_ fix these issues is [here](https://github.com/dlang/druntime/pull/3582).

I haven't been able to work on `_d_arrayappendT` and `_d_arrayappendcTX`, but I plan to catch up next week.

Thanks,\
Teodor
