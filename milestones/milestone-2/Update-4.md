# [SAoC 2021] Replace druntime Hooks with Templates: Milestone 2, Week 4

This week I focused on finishing the replacement of the `_d_arrayctor` hook with a template.
After receiving the advice to void-initialise the array created by `_d_arrayctor`, I changed its signature so that lowerings to it now make use of its return value, via [this PR](https://github.com/dlang/druntime/pull/3611).

The reasoning behind this change was outlined in [this forum post](https://forum.dlang.org/post/simesvkancmscrtsciwq@forum.dlang.org).
Long story short, `_d_arrayctor` was and still is strongly pure.
Because expressions such as `T[4] a = b` were lowered to `_d_arrayctor(a[], b[])`, this meant that the lowering was a call to a strongly pure function whose return value was ignored.
This made the compiler issue some warnings, while also running the risk of having the compiler remove the call to `_d_arrayctor` altogether. 

Next, I changed the aforementioned lowering to `a = _d_arrayctor(!typeof(a))(b[])`.
This worked, except for [this test](https://cirrus-ci.com/task/4617199160655872?logs=test_druntime#L1548), where I noticed that the compiler inserts a `catch` block at the end of `_d_arrayctor`.
This block catches the exception thrown [here](https://github.com/dlang/druntime/blob/1a2f2158345ece9fc3eec098a55519644bf6dfd0/src/core/internal/array/construction.d#L73), calls `__ArrayDtor` on `to` and then re-throws the exception.
But the first elements of the array have already been destroyed by `_d_arrayctor` itself.
This creates something like a double free error.
In addition, `__ArrayDtor` calls the dtor of each element, including the final ones, which have not been initialised.
I am currently looking for ways to remove this final catch block without negatively impacting use cases when this block really is needed.
