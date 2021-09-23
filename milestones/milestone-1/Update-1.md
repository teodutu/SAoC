# [SAoC 2021] Replace druntime Hooks with Templates: Milestone 1, Week 1

Hi,

My project wants to replace dmd hooks to druntime functions such as `_d_arrayctor`, `_d_arrayappendT` and `_d_arraycatT` with templates.
This will make lowerings to these functions faster at runtime, as their templated versions will not require all the indirection used by the hooks.
Speaking of indirection, at the end of this project, the `TypeInfo` class will be removed as all its information will be used at compile time, when instantiating the aforementioned templates.
As a result, the maintainability of the compiler will be improved as lowerings will now take place in the semantic analysis phase, as opposed to the IR phase (where the hooks are currently present).

Dan Printzell worked on this in the past, as part of his [GSoC 2019 project](https://github.com/Vild/GSOC2019).
I'll be picking up the work that he left and then I'll continue with the remaining hooks.

During my first week, I worked on finishing Dan's [PR](https://github.com/dlang/dmd/pull/10102) for the `_d_array{,set}ctor` functions:
- My mentors (Eduard Stăniloiu and Răzvan Nițu) and I fixed some of [Dan's logic](https://github.com/dlang/dmd/blob/e7c81c3021f5948a2a743c0926969f5370178f28/src/dmd/expressionsem.d#L9008-L9016) for choosing when to perform the lowering or not.
We skipped lowering in expressionsem.d for snippets like `S[3] arr = [S(1), S(2), S(3)];`, as e2ir.d already handles such cases more elegantly.
- The tests that are currently failing are runnable/testassign.d and compilable/interpret3.d.
For example, runnable/testassign.d fails because in expressions like `funcVal(a = b)` the temporary variable used as the actual argument for `funcVal` is not lowered to use `_d_arraysetctor`, when it actually should.
I am still working on fixing it.

Thanks,
Teodor
