# [SAoC 2023] Replace DRuntime Hooks with Templates Weekly Update #1

Hi,

I started this week debugging [my older PR for converting `_d_newarray{U,iT,T}` to two templates](https://github.com/dlang/dmd/pull/15299):

- `_d_newarrayT` is the template to which the compiler lowers `new A[n]`.
It decides how to initialise the elements of the array based on their type at compile time.

- `_d_newarrayU` is the generic implementation called by `_d_newarrayT`.
It is necessary to keep this hook separate because other functions such as `dup()` call `_d_newarrayU`.
They do so because there's no need to initialise the copied array since its elements will be copied from the older array.

I couldn't remove the old hooks from `rt/lifetime.d` because they're still used by `_d_newarraym{i,}TX`.
I'll be able to remove the old `_d_newarray*` hooks when I convert `_d_newarraym` to templates, hopefully in the following weeks.

I tried updating `dup()` to also use the template `_d_newarrayU`, but this resulted in some errors that became difficult to manage given the other changes required for this PR.
I intend to take care of `dup()` after the `_d_newarray{U,iT,T}` hooks are converted to templates.
Up to now the latter are passing all tests (except for the FreeBSD ones that seem to be broken globally) and I'm looking forward to continuing with `_d_newarraym*` and `dup()`.

Thanks,\
Teodor
