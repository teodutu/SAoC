# [SAoC 2022] Replace druntime Hooks with Templates: Milestone 2, Week 4

Hi,

I spent this week looking into the template `_d_newitemU`.
What's most difficult is inserting code to initialise `vthis` and `vthis2` in the frontend.
`e2ir.d` already has machinery for this in place, but moving this logic to the frontend is more tedious than I originally thought.

I did talk to my mentors about this and Razvan suggested we add a new AST node to the frontend to which we delegate copying `vthis`.
The good thing regarding this approach is that it is going to be cleaner in DMD since it won't requier reinventing the wheel to move handling of context pointers to the frontend.
However, this will definitely cause trouble to LDC and GDC as they are going to have to handle this new AST node on their own.
Thus the lowering will no longer be transparent to the other compilers.

My mentors and I are still discussing this issue.
We will probably bring it up for debate with the rest of the community if we can't find a solution to satisfy all needs.

In the meantime, I spent some time preparing other hooks such as `_d_newarray*` and `_d_newclass`, but ultimately they are going to be blocked by the same issue regarding context pointers, so this decision cannot wait too much.

In the following weeks, we aim to reach a decision so as to move past this blocker and continue working on the rest of the hooks.

Thanks,\
Teodor
