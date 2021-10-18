# [SAoC 2021] Replace druntime Hooks with Templates: Milestone 1 Report
This project aims to replace DMD hooks to druntime with template lowerings, as described [here](https://github.com/dlang/projects/issues/25).
Of particular importance is the work of Dan Printzell ([Vild](https://github.com/Vild/)), which this project aims to complete.
[This comment](https://github.com/dlang/projects/issues/25#issuecomment-591866063) outlines the basic workflow that is necessary in order to convert a hook to a template function.

## Week 1
I started the first week by rethinking the situations in which lowerings to `_d_arrayctor` are performed.
In particular, when the rhs operator is an array literal, lowerings are no longer performed, as the backend handles such cases more elegantly by assigning elements individually instead of calling a function.
Furthermore, there was an issue with lowerings also being performed on rvalues and references.
This has also been removed, as these should be moved, not copied.

## Week 2
During the second week, I finished the [PR](https://github.com/dlang/dmd/pull/13116/files) to change the `_d_arrayctor` and `_d_arraysetctor` hooks to templates.
This involved updating the error messages for some tests and refactoring the code to merge similarly looking code branches.
After creating this PR, I started working on fixing the then-failing druntime tests.

Those tests were failing because `_d_arrayctor` was not `@nogc`, due to its calls to `enforceRawArraysConformable`, which is not `@nogc`.
As a result, I created [this PR](https://github.com/dlang/druntime/pull/3582) to change
the error handling in `_d_arrayctor` so that it no longer used any array concatenation.
This allowed `_d_arrayctor` to remain `@nogc`.
As a side effect, I had to simplify the error messages to only display some static, predefined text.

## Week 3
My mentor Edi [suggested](https://github.com/dlang/druntime/pull/3582#discussion_r725480441) that I treat the aforementioned error messages similarly to how `staticError`s are handled.
So I went on and [implemented Edi's suggestion](https://github.com/dlang/druntime/pull/3583).

During this week, however, my mentors and I realised that the strong purity that enforced to `_d_arrayctor` was stabbing us in the back in cases where the function was instantiated on `const` or `immutable` types.
In a snippet like the one below, `immutable int[2] b = a;` is lowered to `_d_arrayctor(b, a)`.
```d
immutable int[2] a;
immutable int[2] b = a;
```
But its return value is ignored and, being a pure function, the compiler believes the call to it has no effect and thus issues warnings (which is why the DMD PR above failed the phobos tests).
It might also be inclined to remove this call altogether, which might be disastrous.

## Week 4
In light of the observations from the previous week, I created [this PR](https://github.com/dlang/druntime/pull/3587) to implement a workaround to convert `_d_arrayctor`'s strong purity into weak purity.
Since this workaround is, in essence, a hack, I asked [this question](https://forum.dlang.org/post/simesvkancmscrtsciwq@forum.dlang.org) on the forum.

In addition, the failure of [this test](https://github.com/dlang/druntime/blob/b8f47bcb00435fb11a206bb5356f0bafb570641f/src/core/internal/postblit.d#L273) revealed an old bug in the compiler's backend whereby the optimiser did not take `try / catch` blocks into account when eliminating variables during optimisation.
I [filed an issue](https://issues.dlang.org/show_bug.cgi?id=22372) and wrote a [forum post](https://forum.dlang.org/post/tfovzyyscbuimlthpeci@forum.dlang.org) about this topic, which led to the bug being fixed.

I am currently trying a different approach to fixing the aforementioned warnings issued by the compiler for some calls to `_d_arrayctor`.
This approach is to refactor `_d_arrayctor` to use two template types instead of one:
```d
Tarr1 _d_arrayctor(Tarr1 : T1[], T1, Tarr2 : T2[], T2)(return scope Tarr1 to, scope Tarr2 from)
```
This new signature still produces some instantiation errors, which I'm investigating.

## Milestone 2
During the next milestone, I will:
- pick up converting the `_d_arrayappendT` and `_d_arrayappendcTX` hooks to templates again, starting from [Dan's previous work](https://github.com/dlang/dmd/pull/9982/).
I spent a short time on these hooks during the second week, which helped me track down some bugs, which I'll be trying to fix during the following weeks.
- implement a definitive and more elegant solution to the warnings caused by `_d_arrayctor`'s strong purity.

## Weekly Forum Posts
- [Week1](https://forum.dlang.org/post/bbhmgddyabnuxajlxgqp@forum.dlang.org)
- [Week2](https://forum.dlang.org/post/pxmvkkhbjahauldjezdn@forum.dlang.org)
- [Week3](https://forum.dlang.org/post/hqaobmhwepknxaoiejsc@forum.dlang.org)
- [Week4](https://forum.dlang.org/post/eiblybnzjygeueiqgxmr@forum.dlang.org)
