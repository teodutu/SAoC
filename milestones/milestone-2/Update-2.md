# [SAoC 2021] Replace druntime Hooks with Templates: Milestone 2, Week 2

Hi,

This week I made some progress with regards to the conversion of `_d_arrayappendcTX` from using a hook to a template.
This week I doubled back on last week's decision to replace [this `assert`](https://github.com/dlang/dmd/blob/a890f134af71c553d74ea346650cdea1d2e5f0ad/src/dmd/dinterpret.d#L5038-L5041) in order to convert the call to `_d_arrayappendcTX(ea, eb)` to `ea.length += 1`.
Razvan and I reached this decision based on [these comments](https://github.com/dlang/dmd/pull/9982/files#r310356331).
They showed us that Dan's initial implementation was actually correct in lowering `a ~= b`
to `__ctfe ? a ~= b : _d_arrayappendcTX(a, 1), a[$ - 1] = b, a;` and not just to the latter comma expression.
The reason for this particular lowering is so that dinterpret.d is can use `__ctfe` to interpret `a ~= b` as a whole.
Moreover, during the inline phase, this conditional expression is replaced with an equivalent `if - else` statement.
This also passes the `a ~= b` branch to be backend, which triggered [this asert](https://github.com/dlang/dmd/blob/a890f134af71c553d74ea346650cdea1d2e5f0ad/src/dmd/e2ir.d#L3297)
So the solution was to remove the `if` body and only keep the `else` body in s2ir.d.

In the meantime, I've also been working on changing `_d_arrayctor`'s so that it returns the newly created array instead of receiving it as a parameter.
I've made some progress, but my current implementation allocates the new array dynamically.
Thus, it doesn't make use of NRVO yet and the returned array has to be copied back to its intended destination.

Thanks,
Teodor
