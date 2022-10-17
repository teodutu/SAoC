# [SAoC 2022] Replace druntime Hooks with Templates: Milestone 1, Week 4

Hi,

This week I finished lowering `CatExp`s to `_d_arraycatnTX` and created [this](https://github.com/dlang/dmd/pull/14550) PR for it.
I still have a few small yet annoying errors to fix, regarding mangling and a failure to build phobos that I cannot reproduce locally.
I'll work with my mentors to try to solve these bugs, as they seem rather small and they probably don't require many changes to the PR.

In addition, I made further progress with `_d_newitem{T,iT,U}`.
I am looking to finish translating those hooks to a template (`_d_newitemU`) by the end of this week.

Thanks,\
Teodor
