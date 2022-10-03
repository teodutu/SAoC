# [SAoC 2022] Replace druntime Hooks with Templates: Milestone 1, Week 2

Hi,

This week I faced a few setbacks with `_d_arraycatnTX` due to type mismatches.
Due to the hustle of moving house, I haven't made much progress on this front.

To take a break from this, I temporarily moved on to the `_d_newitem{T,iT,U}` hooks.
I implemented templated one templated hook for `_d_newitemT` and added unittests for it.
The difference will now be made by the compiler during the lowering, by calling the constructor, copying the default initialiser, or doing nothing (`void` initialisation).

This week I am looking to make some progress with this hook and come up with the aforementioned lowerings.

Thanks,\
Teodor
