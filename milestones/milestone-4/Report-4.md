# [SAoC 2021] Replace druntime Hooks with Templates: Milestone 4 Report

## Week 1
I spent the first week of this milestone looking into a bug with the lowering to `_d_arrayappendcTX`.
The lowering of `a ~= b` at that time was
```d
__ctfe ? a ~= b :_d_arrayappendcTX(a, 1), a[$ - 1] = b, a
```
The assignment `a[$ - 1] = b` is internally a `ConstructExp`, so the correct constructor is called.

In the case of a chain of concatenations, such as `(a ~= b) ~= c`, the generated code that resulted from compiling with `-inline` triggered an index out of bounds exception when accessing `$`.

The fact that the hook's functionality doesn't perfectly match that of the expression form which it's lowered would come back to hurt me towards the end of this milestone.
`_d_arrayappendcTX(a, n)` simply extends the array `a` with `n` elements.
This hook was implemented a few years ago and I decided to stick with this implementation and just focus on the compiler.

The reason this lowering is problematic is that, because many druntime hooks call libc functions, they cannot be interpreted at CTFE.
Hence, lowerings to them have to be interpreted as their original expressions.
This is difficult to do when the lowering is a `CommaExp`, which is why I chose to use a `CondExp`.
The `true` branch is the one that CTFE interprets, while the `false` branch gets to the backend.

However, the IR generator receives the entire `CondExp` and needs to discard the `CondExp` and its `true` branch.
As I found out in [week 3](#week-3), this makes lowerings such as this one difficult to implement in other compilers, such as GDC or LDC.

## Week 2
During the second week, not knowing exactly what the problem was with `$` and `_d_arrayappendcTX`, I moved on to lowering `_d_newThrowable`.
I implemented an initial version of the template hook, but, in retrospect, made the mistake of keeping true to its original signature.
Therefore, the templated `_d_newThrowable` still returned a `Throwable` object when, in fact, it could return the instantiated type `T`.

For this reason, `new E(args)` was lowered similarly to `a ~= b`:
```d
__ctfe ? new E(args) : tmp = _d_newThrowable!E(), (cast(E) tmp).__ctor(args), cast(E) tmp;
```

In parallel, I started working on `_d_arraycatnTX`.
This hook corresponds to array concatenations (without assignment): `a ~ b`.
The [older hook](https://github.com/dlang/druntime/blob/33511e263134530a5994a775b03a061ea3f1aa34/src/core/internal/array/concatenation.d#L15-L46) required all concatenated arrays to be grouped in a temporary 2D array, created "behind the scenes" by the compiler.
This adds quite a few errors when performing the lowering.

I changed the hook to take a variadic template argument, but when performing the lowering, I ran into some errors regarding the hook's new signature.
Despite the trouble, I am going to stick to this new signature, as it is much more in line with modern D metaprogramming standards, and keep working on the compiler to get the lowering working.

## Week 3
This week I finished converting `_d_newThrowable` to a template.

Then I fixed the bug with `$` for the lowering to `_d_arrayappendcTX`.
`$` was mishandled by the backend when chained concatenations were inlined.
To mitigate this problem, I used `a.length` instead of `$`.

Following a [discussion](https://github.com/dlang/dmd/pull/13495#discussion_r779559051) on my PR for adding the new lowering to `_d_arrayappend{T,cTX}`, my mentors and I came to the conclusion that I mentioned when describing [week 1](#week-1).

## Week 4
After the discussion from the previous week, my mentors and I decided the best approach would be to [ask the community](https://forum.dlang.org/post/dcwxswfamadkdnflkaap@forum.dlang.org) about how to proceed with those lowerings.
The responses matched our expectations, arguing for a new template hook that would be equivalent to `a ~= b`.
By extension, we now believe all hooks should behave like the expressions from which they are lowered.
This adds some more compile-time logic to the hooks, but heavily simplifies the compiler.

In the future, I will change `_d_arrayappendT` to make it append its second argument to the first, both when the second argument is another array and when it's a single element.
This way, `a ~= b` will lower to `_d_arrayappendT(a, b)` regardless of whether `b` is a single element or another array.
Furthermore, this new lowering will remove the need for the `__ctfe ? ...` trick.
This, in turn, will simplify the code that performs the lowering in the semantic analyser, and will remove the need for the IR generator to trim a part of this lowering.

## SAoC Overview
### What I've Learned
As someone relatively new to D and its ecosystem (I first came into contact with the language by attending last summer's [D Summer School](https://ocw.cs.pub.ro/courses/dss), held by my mentors Razvan and Edi), the D compiler has been the largest and longest-living project I've ever worked on.
It's also been the one on which I've been working for the longest amount of time.

Having taken a course on compilers last year at university, I had some previous experience with them.
The course introduced me to the general layout of a compiler, its different phases, the need for a frontend and a backend, the use of the visitor pattern, basic type deduction etc.
This gave me a false sense of certainty at the beginning of the project.
DMD looked similar to what I already knew about compilers, but when I had to actually make changes to it and then debug my changes, it proved to be something entirely different.
At this point, I realised how little I truly knew about compilers (basically the downwards slope of the Dunning–Kruger effect).

At this point I had the chance to learn about DMD's inner workings from my mentors, who taught me a lot about the compiler's internal flow.
They also introduced me to the D community, encouraging me to write forum posts and take part in the subsequent discussions when we couldn't find solutions for problems ourselves or when we were uncertain about our approach and wanted to hear some other opinions.

So I'd say one of my main takeaways from SAoC 2021 has to be code literacy.
I feel that now I am much more capable of diving into a large codebase and making sense of what's going on there than I was before SAoC.
On the technical side, I became a lot more knowledgeable about some more advanced topics regarding compilers, such as NRVO and CTFE, as well as some related subjects, like garbage collection and function purity.

### Overcoming Problems
I think the most interesting technical problem I had to overcome was back when I was working on converting `_d_arrayctor` to a template.
It was called when lowering an array construction, such as `S[3] a = b`, like so: `_d_arrayctor(a[], b[])`.
Pretty straightforward.
However, after fixing every other bug, there remained one, where some code in phobos would issue warnings because the hook was evaluated to be pure, but its return value was unused.
But the hook has to return the destination array so that you can chain constructions of static arrays in contexts such as postblits.

This looked like a conundrum until my mentors pointed out to me that the concept of purity is not as straightforward as I thought and that there are two types of it: strong and weak purity.
Our solution was to add an unused, pointer-type parameter to the hook so that it switches from being strongly pure to [being weakly pure](https://github.com/dlang/druntime/pull/3587).

Unsure about this rather unorthodox approach, we [asked the community](https://forum.dlang.org/post/hajlsppmugslhinluzos@forum.dlang.org) about it.
We also provided [metrics](https://imgur.com/s9JjXSi) to support our approach.
They clearly showed that making the hook weakly improves its performance by about 25% when compared to the original hook.
And the community accepted our claim and we went on with it.
In the end, what looked like a crazy hack turned out to be a reasonable and I'd say rather cool workaround.
I plan on using a similar approach when converting `_d_arrayassign{,_l,_r}` to templates, as they are very similar to `_d_arrayctor`.

### Plans for the Future
I am a first-year MSc student and replacing druntime hooks with templates is part of my dissertation project.
It aims to allow D applications to run on microcontrollers.
Currently, you can do this by compiling them with `-betterC`, but this strips much of the D's features, such as classes, garbage collection and array concatenation.

This is where my project comes in.
By converting DRuntime (or as much of it as possible) to templates, it wouldn't have to be statically linked against every D executable and would become a template library, somewhat similar to C++'s header-only libraries.
Therefore, I will continue the work I started during this project.

My immediate goals are to get the PRs for `_d_arrayappend{T,cTX}` and `_d_newThrowable` merged.
Then I will pick up where I left the lowering to `_d_arraycatnTX`.
Some hooks like `_d_newclass` and `_d_newitem{U,iT,T}` are likely to be very similar to `_d_newThrowable`, while others like `_d_arrayassign{,_l,_r}` will resemble `_d_arrayctor`.

I believe I have learned enough by now to be able to move faster with the next hooks.

## Weekly Forum Posts
- [Week 1](https://forum.dlang.org/post/npzmxavjervvgujhkbxv@forum.dlang.org)
- [Week 2](https://forum.dlang.org/post/npkacnzbfkbmvpnoivoh@forum.dlang.org)
- [Week 3](https://forum.dlang.org/post/flwjoxilxfpvdjheehdg@forum.dlang.org)
- [Week 4](https://forum.dlang.org/post/bdtnuozkxvroqkcxbkfq@forum.dlang.org)
