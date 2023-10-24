# [SAoC 2023] Replace DRuntime Hooks with Templates Weekly Update #5

Hi,

This week I took a deeper look into my fix for [this bug](https://issues.dlang.org/show_bug.cgi?id=24159).
It involves storing the lowering in a new field within `CatAssignExp` (called `lowering`).
It previously ran into a backend error that turned out to be caused by incorrectly inlining the lowering instead of the `CatAssignExp`.
The lowering could not be inlined as often as the original expression.
With this out of the way, I am now facing another error during CTFE as it cannot interpret the lowering.
But similarly to the backend error from earlier, CTFE shouldn't interpret this lowering.
Now I am trying to skip this step and interpret the right thing instead.

In addition, the [PR for converting `_d_newarray{U,iT,T}` to templates](https://github.com/dlang/dmd/pull/15299) was merged and a discussion arose about including it in the changelog.
I'm not sure about this, so I created [this PR](https://github.com/dlang/dmd/pull/15729) to add a changelog entry about the template `_d_newarrayT`.
If you have any opinions on this, let's discuss them there.

Thanks,\
Teodor
