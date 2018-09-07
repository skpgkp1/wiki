### How to find exact switch for gcc different optimization label O3,O2 or O1.

#### In order to successfully debug gcc compiled runtime crash one need to figure out minimal set of options to reproduce the issue.

https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html

```
$ gcc --version
gcc (GCC) 9.0.0 20180907 (experimental)
Copyright (C) 2018 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.


$  gcc -O3 -Q --help=optimizers -xc - < /dev/null  > /tmp/O3
$  gcc -O2 -Q --help=optimizers -xc - < /dev/null  > /tmp/O2
$  gcc -O1 -Q --help=optimizers -xc - < /dev/null  > /tmp/O1

$ sdiff -s /tmp/O3 /tmp/O2
  -fgcse-after-reload                   [enabled]             |   -fgcse-after-reload                   [disabled]
  -finline-functions                    [enabled]             |   -finline-functions                    [disabled]
  -fipa-cp-clone                        [enabled]             |   -fipa-cp-clone                        [disabled]
  -floop-interchange                    [enabled]             |   -floop-interchange                    [disabled]
  -floop-unroll-and-jam                 [enabled]             |   -floop-unroll-and-jam                 [disabled]
  -fpeel-loops                          [enabled]             |   -fpeel-loops                          [disabled]
  -fpredictive-commoning                [enabled]             |   -fpredictive-commoning                [disabled]
  -fsplit-loops                         [enabled]             |   -fsplit-loops                         [disabled]
  -fsplit-paths                         [enabled]             |   -fsplit-paths                         [disabled]
  -ftree-loop-distribute-patterns       [enabled]             |   -ftree-loop-distribute-patterns       [disabled]
  -ftree-loop-distribution              [enabled]             |   -ftree-loop-distribution              [disabled]
  -ftree-loop-vectorize                 [enabled]             |   -ftree-loop-vectorize                 [disabled]
  -ftree-partial-pre                    [enabled]             |   -ftree-partial-pre                    [disabled]
  -ftree-slp-vectorize                  [enabled]             |   -ftree-slp-vectorize                  [disabled]
  -funswitch-loops                      [enabled]             |   -funswitch-loops                      [disabled]
  -fvect-cost-model=[unlimited|dynamic|cheap]   dynamic       |   -fvect-cost-model=[unlimited|dynamic|cheap]   cheap


$ sdiff -s /tmp/O2 /tmp/O1
  -falign-functions                     [enabled]             |   -falign-functions                     [disabled]
  -falign-functions=                    16                    |   -falign-functions=
  -falign-jumps                         [enabled]             |   -falign-jumps                         [disabled]
  -falign-jumps=                        16:11:8               |   -falign-jumps=
  -falign-labels                        [enabled]             |   -falign-labels                        [disabled]
  -falign-labels=                       0:0:8                 |   -falign-labels=
  -falign-loops                         [enabled]             |   -falign-loops                         [disabled]
  -falign-loops=                        16:11:8               |   -falign-loops=
  -fcaller-saves                        [enabled]             |   -fcaller-saves                        [disabled]
  -fcode-hoisting                       [enabled]             |   -fcode-hoisting                       [disabled]
  -fcrossjumping                        [enabled]             |   -fcrossjumping                        [disabled]
  -fcse-follow-jumps                    [enabled]             |   -fcse-follow-jumps                    [disabled]
  -fdevirtualize                        [enabled]             |   -fdevirtualize                        [disabled]
  -fdevirtualize-speculatively          [enabled]             |   -fdevirtualize-speculatively          [disabled]
  -fexpensive-optimizations             [enabled]             |   -fexpensive-optimizations             [disabled]
  -fgcse                                [enabled]             |   -fgcse                                [disabled]
  -fhoist-adjacent-loads                [enabled]             |   -fhoist-adjacent-loads                [disabled]
  -findirect-inlining                   [enabled]             |   -findirect-inlining                   [disabled]
  -finline-small-functions              [enabled]             |   -finline-small-functions              [disabled]
  -fipa-bit-cp                          [enabled]             |   -fipa-bit-cp                          [disabled]
  -fipa-cp                              [enabled]             |   -fipa-cp                              [disabled]
  -fipa-icf                             [enabled]             |   -fipa-icf                             [disabled]
  -fipa-icf-functions                   [enabled]             |   -fipa-icf-functions                   [disabled]
  -fipa-icf-variables                   [enabled]             |   -fipa-icf-variables                   [disabled]
  -fipa-ra                              [enabled]             |   -fipa-ra                              [disabled]
  -fipa-sra                             [enabled]             |   -fipa-sra                             [disabled]
  -fipa-vrp                             [enabled]             |   -fipa-vrp                             [disabled]
  -fisolate-erroneous-paths-dereference         [enabled]     |   -fisolate-erroneous-paths-dereference         [disabled]
  -flra-remat                           [enabled]             |   -flra-remat                           [disabled]
  -foptimize-sibling-calls              [enabled]             |   -foptimize-sibling-calls              [disabled]
  -foptimize-strlen                     [enabled]             |   -foptimize-strlen                     [disabled]
  -fpartial-inlining                    [enabled]             |   -fpartial-inlining                    [disabled]
  -fpeephole2                           [enabled]             |   -fpeephole2                           [disabled]
  -freorder-blocks-algorithm=[simple|stc]       stc           |   -freorder-blocks-algorithm=[simple|stc]       simple
  -freorder-blocks-and-partition        [enabled]             |   -freorder-blocks-and-partition        [disabled]
  -freorder-functions                   [enabled]             |   -freorder-functions                   [disabled]
  -frerun-cse-after-loop                [enabled]             |   -frerun-cse-after-loop                [disabled]
  -fschedule-insns2                     [enabled]             |   -fschedule-insns2                     [disabled]
  -fstore-merging                       [enabled]             |   -fstore-merging                       [disabled]
  -fstrict-aliasing                     [enabled]             |   -fstrict-aliasing                     [disabled]
  -fthread-jumps                        [enabled]             |   -fthread-jumps                        [disabled]
  -ftree-pre                            [enabled]             |   -ftree-pre                            [disabled]
  -ftree-switch-conversion              [enabled]             |   -ftree-switch-conversion              [disabled]
  -ftree-tail-merge                     [enabled]             |   -ftree-tail-merge                     [disabled]
  -ftree-vrp                            [enabled]             |   -ftree-vrp                            [disabled]
  -fvect-cost-model=[unlimited|dynamic|cheap]   cheap         |   -fvect-cost-model=[unlimited|dynamic|cheap]   [default]

```
