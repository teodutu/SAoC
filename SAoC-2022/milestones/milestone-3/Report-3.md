# [SAoC 2022] Replace druntime Hooks with Templates: Milestone 3 Report

## Week 1

This week I looked into some [extra lines](https://github.com/dlang/dmd/blob/9c6e6b3f95a9cc9b873a4d61b58a5e6f0e8101e3/druntime/test/profile/myprofilegc.log.linux.64.exp#L2) produced by running `-profile-gc` tests for [`_d_arraycatnTX`](https://github.com/dlang/dmd/pull/14550).
After getting some opinions from the community via [this forum post](https://forum.dlang.org/thread/uyjolkupyabdilczdain@forum.dlang.org), it turned out that my implementation of `_d_arraycatnTXTrace` was buggy.
The bug caused it to allocate unnecessary bytes which ended up displayed in the `-profile-gc` log files.

## Week 2

Then I tried another approach to copy the context pointer to nested `struct`s created by `_d_newitemT`.
The idea was to create an object in the frontend from which to copy the context pointer inside the runtime hook.
This was similar to the way [`copyEmplace()` handles context pointers](https://github.com/dlang/dmd/blob/a90ed729205e6faeb2803fbc70733d5ec6d58701/druntime/src/core/lifetime.d#L1274-L1280).
Therefore, after consulting the community via in [this forum post](https://forum.dlang.org/thread/mdybepujnpglwqyztnya@forum.dlang.org), the lowering was going to look like this:

```d
// original expression:
S* s = new S(args);

// lowering:
S* s = (void[0] ctxPointerTmp, _d_newitemT!S(ctxPointerTmp))
```

## Week 3

Upon receiving feedback from [Johan](https://forum.dlang.org/post/wrtfsjheteinguxmfprv@forum.dlang.org), [Iain and Dennis](https://github.com/dlang/dmd/pull/14550#discussion_r1034661632), I changed my approach to nearly all lowerings.
If previously, the original AST node was replaced with a new node that contained the lowering, now the original node receives a new field that stores this lowering.
This makes both the original expression and the lowering reach the glue layer, which will now ignore the original expression and "switch" to the lowering.

This approach brings at least 3 benefits:

- not having to handle the call to the runtime hook separately during CTFE
- leveraging some optimisations in LDC and GDC that [Iain pointed out](https://forum.dlang.org/post/lncqlesvnjjtxxlydxbw@forum.dlang.org)
- using the already existing code in `e2ir.d` to copy context pointers for nested `struct`s / classes.

During this week, I also discovered [this bug](https://issues.dlang.org/show_bug.cgi?id=23534), which blocked me for a short while, before Razvan fixed it.

## Week 4

This week I started working on `_d_newclass`, while looking for fixes for _d_arraycatnTX` and `_d_newitemT`.
I am getting close to finishing the work on this hook as well.
I only have about 2 tests left to fix and they are both about spurious errors that should only be printed for the original expression, but are now also printed for the lowering.

It will be pretty tricky to get rid of the old hook, however.
It is being used explicitly by [`Object.create()`](https://github.com/dlang/dmd/blob/2655129c79db15a16ab371d1c723e0fd3334cdbb/druntime/src/object.d#L1690).
I raised this issue in [this forum post](https://forum.dlang.org/post/fjkuvtklgvaamebwxvoj@forum.dlang.org) an [Vladimir suggested](https://forum.dlang.org/post/tkeooauunsdhvtkojjnn@forum.dlang.org) that I use [`toType`](https://dlang.org/spec/traits.html#toType) to get the type of the object form its mangled name at compile time.
I'll try to follow Vladimir's advice as this would allow me to delete the old hook altogether.

## Milestone 4

Next 2 weeks I'll be on holiday from university and I'll have more time to work on my existing PRs as well as to wrap up `_d_newclass`.
Then I hope I'll be able to convert `_d_newarray*` to templates as well.

## Weekly Forum Posts

- [Weeks 1 & 2](https://forum.dlang.org/post/ivowjokzuvfoiwmpzcam@forum.dlang.org)
- [Week 3](https://forum.dlang.org/post/heqpvlnjykzzkfylpwsy@forum.dlang.org)
- [Week 4](https://forum.dlang.org/post/sckludqccsdsuyhceumr@forum.dlang.org)
