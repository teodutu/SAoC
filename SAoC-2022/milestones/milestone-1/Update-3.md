# [SAoC 2022] Replace druntime Hooks with Templates: Milestone 1, Week 3

Hi,

This week, with the help of my mentors, I fixed the type mixmatches with `_d_arraycatnTX`.
Then I fixed the bugs in my lowering and hook so that now all tests in druntime and phobos pass.
The only failing test in dmd is [`test19688.d`](https://github.com/dlang/dmd/blob/81f5c8b354aed2dc53a45e52498dc23f2f40fe88/compiler/test/runnable/test19688.d).
The reason it fails is that passing `__FUNCTION__` as an argument to another function makes it evaluate to an empty string, like in the code below:

```d
string foo(string arg)
{
    return arg;
}

T fooT(T)(T arg)
{
    return arg;
}

void bar(string s = fooT(__FUNCTION__))
{
    assert(s != "", s);  // this fails
}

void baz(string s = foo(__FUNCTION__))
{
    assert(s != "", s);  // this fails
}

void taz(string s = __FUNCTION__)
{
    assert(s != "", s);  // this passes
}

void main()
{
    taz();
    baz();
    bar();
}
```

I am not sure whether this is a bug or not and I am still investigating the issue.
Once I figure it out and fix it or file a bug report, I'll raise a PR with the new `_d_arraycatnTX` hook.
Then I'll continue with `_d_newitem{T,iT,U}`.

Thanks,\
Teodor
