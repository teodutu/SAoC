# [SAoC 2022] Replace druntime Hooks with Templates: Milestone 4 Report

I spent this milestone working on my existing PRs as they are all close to being ready for merging.

First, after moving the template lowering of `CatExp`s to a new field in the `CatExp` class, the compiler started throwing a backend error when compiling the code below (reduced from [`time.d`](https://github.com/dlang/dmd/blob/2e196aedeae6f66da7df9e49ab93134a885db7f1/druntime/src/core/time.d)) with `-inline`:

```d
private struct Tmp(ubyte N)
{
    private char[N] _buf = void;
    char[] get()
    {
        return _buf[0..$];
    } 
}

auto foo() @safe
{
    Tmp!20 result = void;
    return result;
}

string toString() pure nothrow
{
    return foo().get() ~ 'x';
}
```

The lowered call to `_d_arraycatnTX` isn't inlined properly and I still haven't been able to figure out how or where.

The bug preventing the [template `_d_newitemT` to be merged](https://github.com/dlang/dmd/pull/14664) can be reproduced via this code:

```d
__gshared int x;

void main()
{
    static assert(!__traits(compiles,
    {
        static struct S { int *p = &x; }
        immutable s = new S;
    }));
}
```

The test compiles successfully, but then runs into an undefined reference error from the linker because the `TypeInfo` of `S` is generated although it shouldn't.
The reason for this is that the template hook uses `typeid()` to get some `GC.BlkAttr`s needed for the allocation.
Semantic analysis now now sees this `typeid()`, naturally it generates a `TypeInfo`.
But because `S` is in a `!__traits(compiles, ...)` block, there is no `S` after generating the object file, which ends up upsetting the linker.
I'm still working with my mentors to try to fix this issue as we can't simply ditch the usage of `TypeInfo` inside the hook because it is required that it be placed [at the end of the allocated memory](https://github.com/dlang/dmd/blob/2e196aedeae6f66da7df9e49ab93134a885db7f1/druntime/src/rt/lifetime.d#L1193).

Finally, I made a brekthrough with `_d_newclass` of all things.
The last bugs I was trying to fix were some different error messages reported by 2 tests in DMD:

1. One of these tests was [`fail_compilation/fail308.d`](https://github.com/dlang/dmd/blob/2e196aedeae6f66da7df9e49ab93134a885db7f1/compiler/test/fail_compilation/fail308.d).
It was reporting that `TestType` is an infinite type.
Because of the new hook, the new error was not caused by `object.RTInfo!(TestType)`, but by `core.internal.traits.isAggregateType!(MinHeap!(TestType))` (called from within the template hook).
I reached the conclusion that leaving the test unmodified was impossible since `object.RTInfo!(TestType)` is no longer called by the new hook.
Luckily, the call to `new MinHeap!(TestType)()` was unnecessary.
Simply instantiating an object of type `MinHeap!(TestType)` is enought to trigger the error, so I removed the initialisation of this object.

1. The second test was actually incorrect, as described in [this bug report](https://issues.dlang.org/show_bug.cgi?id=23639).
After Razvan fixed this, the [template `_d_newclass` PR](https://github.com/dlang/dmd/pull/14837) is now ready for merging.

## SAoC Overview

I was expecting this SAoC to be quite similar to the previous one, but in many ways it was a novel experience.
Yes, the project was the same, but the bumps I ran into were different.
And so were the things I learned.

### What I've Learned

This year I was more independent than last year and have been able to fix a much greater deal of errors on my own.
Doing so, my understanding of the compiler's code base has improved vastly.
While last year I started with little experience in compilers and learned a great deal of "real-life compiler basics", this year I got more familiar with the statement visitor, context pointers, the GC etc.
The backend is still like Mordor, though.

From a strictly technical perspective, one of the things I learned is a new approach to handling template hooks.
While I was perviously simply replacing the original expression with its lowering in the AST, now I am adding the template lowering as a new field in the original expression's class.
Thanks to [Iain Buclaw's arguments](https://forum.dlang.org/post/lncqlesvnjjtxxlydxbw@forum.dlang.org), I realised that my previous approach was unnecessaringly missing out on certain optimisations in both GDC and LDC.
This greatly simplified the lowering logic of `NewExp`s, as I'm going to describe in the [next section](#overcoming-problems).
In addition, this also removed the need to rewrite the original expression for CTFE since it is kept until the glue layer.

### Overcoming Problems

While last years's challenges had lots to do with the compiler's internals, this year's problems have been more related to my approach in general and have been more conceptual.
One of the main issues revolved around context pointers.
Initially I wanted to copy them to newly allocated `struct`s and classes form the frontend so as to perform the entire lowering of `NewExp`s in the semantic phase.
I spoke to my mentors about this for quite a while.
Having worked on introducing copy constructors in the past, Razvan has struggled with this issue himself and was unable to fix it.
Naturally, he was also very interested in solving it now.
However, while I wish for us to have been able to come up with a _silver bullet_ with which to easily copy context pointers in the frontend, we couldn't find it.

So we resorted to only a "partial" lowering in the frontend by only creating a `CallExp` to the template hooks (`_d_newclass` and `_d_newitemT` so far).
The backend still handles `vthis` and `vthis2` as it did previously.
Now that I think about it, the "partial" lowering makes more sense than our initial plan to handle context pointers in the compier's frontend.
After all, if the glue layer is already able to do this, why reinvent the wheel?
Of course, this still doesn't solve the issue with regards to `struct` copy constructors howerver.
That remains an open question.

### Plans for the Future

Moving on I plan to wrap up these PRs which I feel I am close to fixing and then moving on to the `_d_newarray*` hooks.
I will probably just create a single template to replace all of them and use DBI to account for all previous use cases, like with `_d_newitemT`, but on a larger scale.
Then I guess templates like `_d_dynamic_cast` or `_d_isbaseof` will be easier.
Lastly, I'll have to convert the existing template wrappers on top of non-template hooks, such as [`_d_arraysetlengthT`](https://github.com/dlang/dmd/blob/2e196aedeae6f66da7df9e49ab93134a885db7f1/druntime/src/core/internal/array/capacity.d#L39-L55), to full-fledged templates and remove the old hooks.

Finally, this project is part of my dissertation thesis, which aims to run D applications on microcontrollers.
A more lightweight druntime will be of great help in this endeavour.
