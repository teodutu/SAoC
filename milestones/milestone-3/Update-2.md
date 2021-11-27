# [SAoC 2021] Replace druntime Hooks with Templates: Milestone 3, Week 2

Hi,

This week I ran a benchmark for the 3 possible approaches to `_d_arrayctor`.
They are: the old hook on one hand and two implementations of the new template.
One of these uses a third pointer parameter for `_d_arractor`, in order to convert its strong purity to weak purity, while the other only takes the source array as a parameter and returns the created array.
The latter doesn't use NRVO, thought.
I gave more details about these 3 approaches and abount my benchmark in [this post](https://forum.dlang.org/post/hajlsppmugslhinluzos@forum.dlang.org).

After finishing the benchmark, I resumed work on `_d_arrayappendcTX`.
I'm currently debugging an error where the template is called 3 times instead of one and I'm trying to figure out what code in the compiler is re-run so that this ends up happening.

If I get stuck with `_d_arrayappendcTX`, I'll start looking at `_d_newitem{T,iT,U}`.
These hooks are likely to be simpler to implement because they only call the GC.

Thanks,\
Teodor
