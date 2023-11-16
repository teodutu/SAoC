# [SAoC 2023] Replace DRuntime Hooks with Templates Weekly Update #6

Hi,

Continuing my work on fixing [this bug](https://issues.dlang.org/show_bug.cgi?id=24159), it turned out that I was incorrectly modifying the original expression like this:

```d
arr ~= a + b;

// was lowered to:
__tmp = a + b; _d_arrayappendcTX(arr, 1), arr[$ - 1] = tmp;

// but was also changed to:
arr ~= __tmp;
```

Creating the temporary variable was necessary for situations such as `arr ~= arr[$ - 1]`.
Without it, the lowering would have been `_d_arrayappendcTX(arr, 1), arr[$ - 1] = arr[$ - 1]`.
Note that in the second expression `arr[$ - 1]` is no longer referring to the same element as in the original expression, but to the newly created element, which is uninitialised.

However, the fix was too hard for the problem it was trying to solve.
Simply modifying the `IndexExp` in the lowering from `$ - 1` to `$ - 2` was enough and more efficient at the same time.
This got rid of the backend bug where the `__tmp` variable was unrecognised.

However, now this [function from DRuntime](https://github.com/dlang/dmd/blob/49690857b45f3f378ff84ff12ece51f6248a0303/druntime/src/core/demangle.d#L2141) seems to be visited twice in the glue layer.
This makes its `vthis` to be visited twice as [this `assert`](https://github.com/dlang/dmd/blob/49690857b45f3f378ff84ff12ece51f6248a0303/compiler/src/dmd/glue.d#L949) is triggered.
I haven't found the source of this error yet.

I also spent some time on convering `_d_newarraym{i,}TX` to templates and hope I'll be able to fix the final bugs and create a PR for these hooks by the end of this week.

Thanks,\
Teodor
