# [SAoC 2021] Replace druntime Hooks with Templates: Milestone 1, Week 2

Hi,

This week I finished patching Dan's previous work on the `_d_arrayctor` and `_d_arraysetctor` hooks.
There was no need for a PR to druntime, as the existing code was working fine.
The PR to DMD is [here](https://github.com/dlang/dmd/pull/13116).
There was no need for an additional PR to druntime.

I've also started working on the `_d_arrayappendT` and `_d_arrayappendcTX` hooks.
As usual, my starting point is [this PR](https://github.com/dlang/dmd/pull/9982) form Dan.
My current status is:
- Trying to fix an error at [this line](https://github.com/dlang/dmd/blob/2defede4b40c63528ad208b7b6dc0089be6cdcac/test/runnable/test21403.d#L13) whereby the compiler generates 3 separate calls to `cat11ret3` when appending the final 7.
- Some tests fail because of linker errors related to the functions I'm working on.
There seems to be an issue with the way they're mangled.

Thanks,\
Teodor
