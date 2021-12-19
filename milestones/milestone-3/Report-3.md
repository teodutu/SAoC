# [SAoC 2021] Replace druntime Hooks with Templates: Milestone 3 Report
During this milestone I settled on [this](https://github.com/dlang/druntime/pull/3587) approach for `_d_arrayctor`, replaced `_d_delstruct` with a template and almost finished doing so for `_d_arrayappend{cTX, T}` as well.

## Week 1
This week I refined a version of `_d_arrayctor` that wasn't using an additional pointer-type parameter in order to become weakly pure.
Instead, it attempted to create the constructed array on the hook's stack and return it, by triggering NRVO.
In order to prevent the compiler from inserting a call to `__ArrayDtor` at the end of the function, which ran te risk of causing a double `free` or of `free`-ing unallocated array elements, I alocated the new array inside a union.
The 2 PRs implementing this version of the hook are here:
-  https://github.com/dlang/druntime/pull/3611
-  https://github.com/dlang/druntime/pull/3627

However, the addition of the union prevented NRVO :(.

## Week 2
As a result of my work from the previous week, I ran a benchmark for the 3 possible approaches to `_d_arrayctor`.
Its results are displayed in [this post](https://forum.dlang.org/post/hajlsppmugslhinluzos@forum.dlang.org).
Since decreasing the running times of hooks is the primary goal of this project, the decision was to go with the weakly pure approach.
In the post, it's called **the hack**.

Then, I resumed the work on `_d_arrayappend` and started investigating an error whereby the hook was called too many times.

## Week 3
This week I fixed the bug I discovered last week.
It turned out that the initial lowering of an expression such as `(a ~= 1) ~= 2` was incorrect.
`a ~= x` was lowered to `__ctfe ? a ~= x : _d_arrayappendcTX(a, 1), a[$ - 1] = x, a`, where `a` was used literally, regardless of what expression it was.
So, if `a` had actually been a function call, for example, that function would have been called 3 times.
Similarly, when `a` was `a ~= 1`, as in the previous example, this concatenation was made multiple times.
Hence, the error.

On another note, this week I started working on `_d_delstruct` and added the template-ised hook to druntime: https://github.com/dlang/druntime/pull/3639.
On the compiler, I changed the lowering of `delete`s to the hook, but ran into the trouble that some of the tests in `fail_compilation/` were displaying different error messages.
These messages were now mentioning the hook, as opposed to the struct's dtor or the `delete` statement itsefl, as they did preivously.
This was unwanted because it displayed information from the compiler's internals to the user, which made errors confusing.

## Week 4
I solved the `_d_arrayappendcTX` error form last week by saving the lhs in a temporary variable whenever this expression isn't a variable, array literal, or something similar.
There are, however, some corner cases to this.
For now, I only look at the rhs expression itself in order to decide whether to use a temporary variable or not.
However, when that expression is, for instance, `a[i]`, it can be used as-is if `opIndexOpAssign` is undefined for `a`.
Otherwise, it should be saved.
I was unable at that time to find a way of figuring out which expressions to store in a temporary variable and which not, without creating an overengineered verification.

On the `_d_delstruct` front, I made [the PR](https://github.com/dlang/druntime/pull/3639) to change the lowering of `delete` to the new hook from the previous week.
However, Razvan [pointed out to me](https://github.com/dlang/dmd/pull/13398#discussion_r764625641) that I had made a mistake by not considering the case when the `-gc_profile` parameter was given.
Hence, I made [another PR](https://github.com/dlang/druntime/pull/3644) to add `_d_delstructTrace` as an alias to `_d_delstruct`, with both of them wrapped in a template.

## Week 5
Initially the lowering to the `_d_delstruct` hook was a `CommaExp` containing both the original and the lowered expressions.
This was so that the `DeleteExp` was checked for `nogc` and `nothrow` erros while the errors from `_d_delstruct` were filtered out.
This approach was inelegant and Razvan taugth me that, despite the downside of duplicating some code, it would be clearer to semantically analyse the dtor of the struct and output errors as if they were for a `delete` statement.
I think that now the lowering to the new `_d_delstruct` can be merged.

I put the dilemmas from last week regarding `_d_arrayappendcTX` on hold and started fixing some more immediate issue: some errors in phobos which I had discovered during the previous week whereby the pure `_d_arrayappendT` cannot call an impure `copyEmplace`.
I might have to lie to the compiler that the hook is `pure`, as I did with [`_d_delstruct`](https://github.com/dlang/druntime/blob/6e12e69b645692109017f0bf3da6ab5d1935a69f/src/core/lifetime.d#L2294-L2295).

## Milestone 4
I am quite close to finishing the new lowering of `_d_arrayappendcTX` and `_d_arrayappendT` so this will be my main objective for this milestone.
In addition, I want to convert as many of the hooks not involving arrays as I can.
These are `_d_newitem{T,iT,U}` and `_d_newThrowable`.

## Weekly Forum Posts
- [Week 1](https://forum.dlang.org/post/jkfcfrphmeyvxcajcctu@forum.dlang.org)
- [Week 2](https://forum.dlang.org/post/jorltjuncqsqdxyoubjq@forum.dlang.org)
- [Week 3](https://forum.dlang.org/post/ljpetarlfpohlaajtnng@forum.dlang.org)
- [Week 4](https://forum.dlang.org/post/sypaebxuatwbjqabpcfu@forum.dlang.org)
- [Week 5](https://forum.dlang.org/post/tdodeirxktmtfkybprov@forum.dlang.org)
