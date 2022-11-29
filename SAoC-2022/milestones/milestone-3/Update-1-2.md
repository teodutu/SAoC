# [SAoC 2022] Replace druntime Hooks with Templates: Milestone 3, Weeks 1 & 2

Hi,

These weeks I fixed the final bugs of the new [`_d_arraycatnTX`](https://github.com/dlang/dmd/pull/14550) hook.
When cleaning up the code, Razvan and I took a closer look at the [extra lines produced by `-profile-gc`](https://github.com/dlang/dmd/blob/9c6e6b3f95a9cc9b873a4d61b58a5e6f0e8101e3/druntime/test/profile/myprofilegc.log.linux.64.exp#L2).
They are referring to the new `_d_arraycatnTX` and seemed to indicate that it allocates some extra memory.
In [this forum post](https://forum.dlang.org/thread/uyjolkupyabdilczdain@forum.dlang.org) I asked how to deal with them.
Following Max's advice, I did some digging and 2 things turned out:

1. the [old hook](https://github.com/dlang/dmd/blob/a90ed729205e6faeb2803fbc70733d5ec6d58701/druntime/src/rt/lifetime.d#L2339-L2373) uses a private function called `__arrayAlloc()` to allocate the concatenated array, which is not instrumented, whereas the new one simply sets the `.length` of the resulting array.
This expression is further lowered to `_d_arraysetlengthT` and this hook [**is** instrumented](https://github.com/dlang/dmd/blob/a90ed729205e6faeb2803fbc70733d5ec6d58701/druntime/src/core/internal/array/capacity.d#L57-L64).
It is possible that this is the cause of the differences.

1. My implementation of `_d_arraycatnTXTrace()` was buggy.
This caused some unnecessary bytes to be allocated.
After fixing this, I'll be able to better look into point 1 from above and be more certain regarding the newly added lines when `-profile-gc` is enabled.

Regarding `_d_newitemU`, we decided it would be more rational to move more of the lowering logic to the hook, as DRuntime already handles [`struct` and context pointer initialisation in `copyEmplace()`](https://github.com/dlang/dmd/blob/a90ed729205e6faeb2803fbc70733d5ec6d58701/druntime/src/core/lifetime.d#L1274-L1280).
I am going to use the same logic in the new `_d_newitemT` (instead of `_d_newitemU`) hook, together with calling the `struct`'s ctor.
What's problematic with this approach is that I need to make the compiler create a "dummy" object before calling `_d_newitemT`, so the lowering would look like this:

```d
// original expression:
S* s = new S(args);

// lowering:
S* s = (void[0] ctxPointerTmp, _d_newitemT!S(ctxPointerTmp))
```

I would like to be able to obtain the context pointer in the frontend of the compiler without having to allocate any other objects.
I asked about this in [this](https://forum.dlang.org/thread/mdybepujnpglwqyztnya@forum.dlang.org) forum.
I am looking forward that the community will help me find a cleaner solution.

Thanks,\
Teodor
