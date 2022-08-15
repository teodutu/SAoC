# Replace `druntime` Hooks with Templates - Planning

The functions that will have to be replaced are listed here:
```bash
druntime $ egrep -o -R "_d_.*\(.*[^;]$" | grep "TypeInfo" | grep -v _d_arraysetlengthT | cut -d ':' -f 2 | egrep -o "_d_[^(]*" | sort -u
_d_arrayappendcTX
_d_arrayappendT
_d_arrayassign
_d_arrayassign_l
_d_arrayassign_r
_d_arraycatnTX
_d_arraycatT
_d_arrayctor
_d_arrayliteralTX
_d_arraysetassign
_d_arraysetcapacity
_d_arraysetctor
_d_arraysetlengthiT
_d_arrayshrinkfit
_d_assocarrayliteralTX
_d_delarray_t
_d_delstruct
_d_isbaseof
_d_newarrayiT
_d_newarraymiTX
_d_newarraymTX
_d_newarrayOpT
_d_newarrayT
_d_newarrayU
_d_newitemiT
_d_newitemT
_d_newitemU
_d_newThrowable
```

I've eliminated the `_d_arraysetlengthT` function as it has been fully
translated into templates by Vild:
- https://github.com/dlang/druntime/pull/2656
- https://github.com/dlang/dmd/pull/10106

In addition, Vild started working on some other functions. These will be my
starting points in order to get used to the type of work needed, as well as to
not reinvent the wheel and make use of already existing code.

As a result, my desired schedule looks as follows:
|          Week           |                                                  Activity / Functions                                               |      Details     |
|:-----------------------:|:-------------------------------------------------------------------------------------------------------------------:|:----------------:|
| 15.09.2021 - 22.09.2021 | `_d_arraysetctor`<br />`_d_arrayctor` (work already started by [Vild](https://github.com/dlang/dmd/pull/10102/))    |
| 22.09.2021 - 29.09.2021 | `_d_arrayappendT`<br />`_d_arrayappendcTX` (will finish [Vild's work](https://github.com/dlang/druntime/pull/2718)) |
| 29.09.2021 - 06.10.2021 | `_d_arraycatnTX` (will pick up where [Vild left](https://github.com/dlang/dmd/pull/10064))                          |
| 06.10.2021 - 13.10.2021 | `_d_arraycatT`<br />**Performance Evaluation**                                                                      |
| 13.10.2021 - 20.10.2021 | `_d_arrayassign`                                                                                                    | First Milestone  |
| 20.10.2021 - 27.10.2021 | `_d_arrayassign_r`<br />`_d_arrayassign_l`                                                                          |
| 27.10.2021 - 03.11.2021 | `_d_arraysetcapacity`<br />`_d_arraysetassign`                                                                      |
| 03.11.2021 - 10.11.2021 | `_d_arraysetlengthiT` <br />`_d_assocarrayliteralTX`<br />**Performance Evaluation**                                |
| 10.11.2021 - 17.11.2021 | `_d_arrayliteralTX`<br />`_d_delarray_t`                                                                            | Second Milestone |
| 17.11.2021 - 24.11.2021 | `_d_delstruct`<br />`_d_arrayshrinkfit`                                                                             |
| 24.11.2021 - 01.12.2021 | `_d_isbaseof`<br />`_d_newarrayiT`                                                                                  |
| 01.12.2021 - 08.12.2021 | `_d_newarraymiTX`<br />`_d_newarraymTX`                                                                             |
| 08.12.2021 - 15.12.2021 | `_d_newarrayOpT`<br />`_d_newarrayT`                                                                                | Third Milestone  |
| 15.12.2021 - 22.12.2021 | `_d_newarrayU`<br />`_d_newitemiT`                                                                                  |
| 22.12.2021 - 29.12.2021 | `_d_newitemT`<br />`_d_newitemU`                                                                                    |
| 29.12.2021 - 05.01.2022 | `_d_newThrowable`<br />**Final Evaluation**                                                                         |
| 05.01.2022 - 15.01.2022 | Cleanup PR                                                                                                          | Final Milestone  |