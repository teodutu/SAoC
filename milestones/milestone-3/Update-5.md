# [SAoC 2021] Replace druntime Hooks with Templates: Milestone 3, Week 5

Hi,

This report also contains the first 3 days of milestone 4, due to the misalignment between weeks and months in the callendar.

This week I finished the lowering of the new hook for `_d_delstruct`.
My first approach for outputing the appropriate errors caused by the `delete` expression was to return a `CommaExp` made up of both the original expression and the lowered call to `_d_arraydtor`.
This was followed by the call when it came to errors and CTFE and ignoring the `delete` in e2ir.d.

However, this was an inelegant solution and Razvan suggested to me that I only return the lowering and gag errors when analysing it semantically.
At the same time, `_d_delstruct` is both `nothrow` and `nogc` because of a hack that was necessary in order for the lowering to be possible.
As a result, when checking the new hook for `nothrow` and `nogc`, only the dtor of the parameter is checked.
Now the code is working without keeping the original `DeleteExp` all the way to e2ir.d.

I also inspected `_d_arrayappendcTX` a bit more.
One solution for my problem from the previous week can be to hack the hook's attribute and lie to the compiler that it's `pure`.
Purity caused the error in question, but this hack might also be necessary for `nothrow` and `@safe`.
However, I can't give a verdict on this yet and still need to explore more options.

Thanks,\
Teodor
