# [SAoC 2021] Replace druntime Hooks with Templates: Milestone 4, Week 1

Hi,

This milestone I am planning to finish replacing the hooks to `_d_arrayappend{T,cTX}` with templates.
This week I almost finished this task.
There is still a bug when inlining a function such as:
```d
int[] arr;
void foo()
{
	(arr ~= 2) ~= 1;
}
```
The first concatenation causes an `ArrayIndexError` because the location of the array length is incorrectly referred in the resulting executable.
I am yet to find the source of this error.
I dissected inline.d, and the resulting statement looks alright.

After this, I plan to convert `_d_newThrowable` and `_d_newitem{U,iT,T}` to templates.
I started working on `_d_newThrowable`, but I'm having trouble recreating [these `assert`s](https://github.com/dlang/druntime/blob/fd9a45448244fb9dd4326520ad8526c540895eb0/src/rt/ehalloc.d#L35-L36) as the template doesn't store those flags.

Merry Christmas!

Thanks,
Teodor
