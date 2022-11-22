# [SAoC 2022] Replace druntime Hooks with Templates: Milestone 1 Report

This milestone has revolved around defining a frontend lowering "methodology" for creating new objects.
This is primarily useful for `_d_newitem{T,iT}`, which are used to create `struct`s on the heap when the developer writes

```d
struct S { ... }
S s = new S(...);
```

`_d_newitemiT` is used when `S` has to be default-initialised, while `_d_newitemT` is used when the `struct` has to be zero-initialised.
To minimise the code size overhead from having 2 template functions, we decided it would be better for the runtime to only contain `_d_newitemU`, which simply allocates a new, **uninitialised** `struct`.
Instead, the compiler would insert the initialisation logic in the frontend.

Successfully converting `_d_newitem{U,T,iT}` to templates will also act as a "template" (not _that_ kind of template though :)) ) for also converting `_d_newclass` and `_d_newarray*` to templates.
As I found out during this milestone, the main obstacle in moving such lowerings to the frontend is copying the context pointers required for nested `struct`s.
This logic will be employed by any runtime hook that allocates objects.

## Week 1

During the first week, I fixed whatever easy bugs were revealed by the CI for lowering [`_d_arraycatnTX` to a template](https://github.com/dlang/dmd/pull/14550).
There still remain some probably compiler-internal bugs, however.

I managed to track one of them down.
[Here](https://github.com/dlang/dmd/blob/9ecf81be49554550202fa61ab5d3ed68c2f37ce0/compiler/test/unit/lexer/location_offset.d#L556) 2 strings are concatenated and then the `.ptr` of the result is used.
Because of the missing `\0` at the end of strings, [this](https://github.com/dlang/dmd/blob/9ecf81be49554550202fa61ab5d3ed68c2f37ce0/compiler/test/unit/lexer/location_offset.d#L563) assert was failing.
I fixed it by manually adding a `~ '\0'` at the end of the concatenation above.

I expect more such cases to be present in the compiler.
This is probably the cause of the [remaining builkite failures](https://buildkite.com/dlang/dmd/builds/28559#0183fc10-644f-4cc9-a285-c96b76ee122a), but such bugs are harder to track.

## Week 2

This week I focused on `_d_newitemU`, specifically getting the template lowering working on non-nested `struct`s so as to avoid the hassle of dealing with `vthis` and `vthis2`.

## Week 3

I continued the work from the previous week to lower `new S(...)` to `_d_newitemU`.
After a few rather hit-or-miss attempts, I reached the following lowering for `new S(args)`:

```d
S tmp = _d_newitemU!S(),
{
    memcpy(&tmp, S.init, S.sizeof),  // for default-initialised allocations
    memset(&tmp, 0, S.sizeof)        // for zero-initialisations
},
tmp.vthis = ...,
tmp.vthis2 = ...
tmp.ctor(args),
tmp
```

Each line above is part of a large `CommaExp` which will be the actual lowering.

## Week 4

Trying to wrap-up `_d_newitemU`, my mentors and I discussed how to handle the context pointers.
Razvan suggested we add a new AST node to the frontend to which we delegate copying `vthis`.
The good thing regarding this approach is that it is going to be cleaner in DMD as it will be able to use the existing machinery in the glue layer that already deals wih context pointers.
But the bad part is that such a solution is unlikely to be welcomed by LDC and GDC as they will have a new node to handle in their own backends.

Previous template hooks had the andvantage of having all their logic in the frontend, thus being available to LDC and GDC "for free".
We're still trying to come up with a solution that ticks both boxes:

- not reinventing the wheel from the backend to the frontend
- being consistent across all D compilers

We're going to ask the community for feedback from the community if we can't find a solution that satisfies all requirements and need to make a compromise.

## Milestone 3

During milestone 3 I plan to finish handling `vthis` for `_d_newitemU` so as to clear the way forward for `_d_newclass` and `_d_newarray*`, which will be the next hooks that I'll convert to templates.
Then I'll try to find a way to fix the last remaining bugs for my older PR that [translates `_d_arraycatnTX` to a template](https://github.com/dlang/dmd/pull/14550).

## Weekly Forum Posts

- [Week 1](https://forum.dlang.org/post/ivowjokzuvfoiwmpzcam@forum.dlang.org)
- [Weeks 2 & 3](https://forum.dlang.org/post/heqpvlnjykzzkfylpwsy@forum.dlang.org)
- [Week 4](https://forum.dlang.org/post/sckludqccsdsuyhceumr@forum.dlang.org)
