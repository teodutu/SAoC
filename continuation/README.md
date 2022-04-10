# `_d_arrayappend{T,cTX}`

I started working on these hooks towards the end of SAoC 2021.
I started with [Vild's original work](https://github.com/dlang/dmd/pull/9982).
I proceeded to fix the bugs form this PR and eventually managed to get it working.

I created a [PR](https://github.com/dlang/dmd/pull/13495) to add the new lowering.
However, a member of the community [pointed out](https://github.com/dlang/dmd/pull/13495#discussion_r780661492) that the lowering is incorrect because it also required code changes to `e2ir.d` and `s2ir.d`.

Following this discussion, I tried to implement a new version of the hooks so that `_d_arrayappendT` can also be used to concatenate an element into an array.
This turned out to be problematic when concatenating a struct literal, such as:
```d
arr ~= S();
// this was lowered to:
_d_arrayappendT(arr, S());
```
The original expression was supposed to not call the postblit of `S` but instead create the new element within the array.
This was, however, impossible because D does not support rvalue references yet, so the call to `_d_arrayappendT` meant an inherent call to the postblit of `S`.

To fix this, I tried to keep using the old hooks, but interpret `_d_arrayappendcTX(arr, n)` at CTFE as `arr.length += n`.
This was impossible for uninitialised associative arrays, such as `int[][string] aai`.
In a scenario like the one below, I managed to initialise `aai` to `["a" : []]` during the first `CatAssignExp`, but triggered some asserts within the compiler during the second concatenation.
```d
int[][string] aai;
aai["a"] ~= 1;
aai["a"] ~= 2;
```
`aai["a"] ~= 2` was lowered to `_d_arrayappendcTX(aai["a"], 1), aai["a"][aai["a"].length - 1] = 2, aai["a"]`.
During CTFE, `_d_arrayappendcTX(aai["a"], 1)` was interpreted as `aai["a"].length += 1`.
The latter expression is invalid during CTFE because of these asserts:
- https://github.com/dlang/dmd/blob/863fec775e2dd8b8f4788849d7b8923b70dcf279/src/dmd/dinterpret.d#L3844
- https://github.com/dlang/dmd/blob/863fec775e2dd8b8f4788849d7b8923b70dcf279/src/dmd/dinterpret.d#L5176

The solution I tried next was to reinterpret the whole `CommaExp` `_d_arrayappendcTX(arr, 1), arr[arr.length - 1] = elem, arr` as `arr ~= elem` instead of only reinterpreting the first expression.
