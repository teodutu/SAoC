# [SAoC 2021] Replace druntime Hooks with Templates: Milestone 2 Report
I spent this milestone finishing my work on the `_d_arrayctor` lowering and making progress to replace the `_d_arrayappendT` and `_d_arrayappendcTX` hooks with templates.

## Week 1
During the first week of this milestone, I resumed the work on `_d_arrayctor`.
The problem being the strong purity of this function (as described in [this forum post](https://forum.dlang.org/post/simesvkancmscrtsciwq@forum.dlang.org)), Razvan suggested that I change its signature so that it now statically creates and returns the destination array.
Thus, the return value of the function would no longer be ignored and the issue with regards to its strong purity solved.
Moreover, this approach was also going to benefit from NRVO, so there would be no performance decrease.

In addition to this change for `_d_arrayctor`, this week I started looking into a CTFE error caused by `_d_arrayappendcTX`, which I fixed during the following week.

## Week 2
As I said above, this week I fixed a CTFE bug created by lowering `a ~= b` to `__ctfe ? a ~= b : _d_arrayappendcTX(a, 1), a[$ - 1] = b, a;`.
The bug was caused by the fact that, during the inlining phase of compilation, the conditional expression was replaced with:
```d
if (__ctfe)
	a ~= b;
else
	_d_arrayappendcTX(a, 1), a[$ - 1] = b;
```
And this code was reaching the backend, where `a ~= b` no longer has an implementation, because the previous hook was removed and replaced with its template counterpart.
The solution for this was to replace the `if-else` statement with just the `else` body, in the s2ir.d.

## Week 3
This week I moved back to `_d_arrayctor` and struggled with being unable to instantiate the templated function using a nested struct.
The reason was that the semantic analysis did not allow the creation and initialisation of the nested struct array on the stack of `_d_arrayctor`.
After doing some research, I believed that this was a bug and [asked about it](https://forum.dlang.org/post/buyuiryhdsgrnjvwisue@forum.dlang.org) on the forum.
It turned out that I was wrong.
As Stanislva Blinov [suggested](https://forum.dlang.org/post/gxtqrukutmkbwlelvfcl@forum.dlang.org), the solution was to void-initialise the returned array.

## Week 4
This week I proceeded to void-initialise the array from last week and then spent time fixing a final bug, whereby the compiler was introducing a call to `__ArrayDtor` to destroy the newly created array, before throwing the would-be exception from `_d_arrayctor`.
This call was potentially harmful because it was destroying all elements of the new array despite not all of them having been initialised.

## Milestone 3
During the first week of this milestone, I completed the new lowering of `_d_arrayctor`.
I solved the above issue by declaring the array returned by `_d_arrayctor` inside a union, so as to avoid the introduction of the call to `__ArrayDtor`.
This trick was confirmed as part of the spec by Adam Ruppe [here](https://forum.dlang.org/post/xqgfaicsqlbbekkbzqye@forum.dlang.org).

For the rest of this milestone, my first priority is to run a benchmark for testing the performance increase when using the new lowerings for `_d_arrayctor` and `_d_arraysetctor`.
Then I plan to finish replacing the lowerings to `_d_arrayappendT` and `_d_arrayappendcTX` with templates, part of this work being started during milestone 2.

In the future (maybe towards the end of this milestone if time allows it, or otherwise for the duration of the last milestone), I'll move away for a while from lowerings involving array manipulation and towards hooks that manipulate somewhat simpler objects, such as `_d_delstruct`, `_d_newitem{T,iT,U}` or `_d_newThrowable`.
I plan to make this shift because I've seen array-based hooks generate some rather time-consuming bugs.
I hope that by doing so, I'll be able to get more work finished and get stuck less.

## Weekly Forum Posts
- [Week 1](https://forum.dlang.org/post/ejhxebtidkrvcznornyn@forum.dlang.org)
- [Week 2](https://forum.dlang.org/post/mfflrbgncvexkqjongmr@forum.dlang.org)
- [Week 3](https://forum.dlang.org/post/vlggwatqlyvhtqhxhimh@forum.dlang.org)
- [Week 4](https://forum.dlang.org/post/kocccsnqxjailweqcazz@forum.dlang.org)
