# [SAoC 2021] Replace druntime Hooks with Templates: Milestone 2, Week 3

Hi,

This week I hit another dead end with regards to changing `_d_arrayctor`'s signature.
More specifically, a signature that _should_ pass all tests as well as make use of NRVO is this one:
```d
Tarr1 _d_arrayctor(Tarr1 : T1[], Tarr2 : T2[], T1, T2)(scope Tarr2 from) @trusted
{
    // ...
    Tarr1 to;
    // ...
    return to;
}
```
However, there seems to be an bug old bug concerning nested structs whereby the compiler doesn't allow me to create `to` when `T1` is a nested struct, as in the example below, taken from [this](https://github.com/teodutu/druntime/blob/69ba1f900733f9929d3f704c9c66d393806a343b/src/core/internal/array/construction.d#L84-L99) unittest:
```d
@safe unittest
{
    int counter;
    struct S
    {
        int val;
        this(this) { counter++; }
    }

    S[4] arr1;
    S[4] arr2 = [S(0), S(1), S(2), S(3)];
    arr1 = _d_arrayctor!(typeof(arr1), typeof(arr2), S, S)(arr2[]);
}
```
I wrote [this](https://forum.dlang.org/post/buyuiryhdsgrnjvwisue@forum.dlang.org) forum post to ask tho community what to do about this bug.
I tried to give as detailed of an account of the situation as I could.
The bug is has been previously reported here, but it seems it's still active:
- https://issues.dlang.org/show_bug.cgi?id=8850
- https://issues.dlang.org/show_bug.cgi?id=8863

As a result, I've switched over to the `_d_arrayappend` hooks once more and am looking forward to progress with those in the near future.

Thanks,\
Teodor
