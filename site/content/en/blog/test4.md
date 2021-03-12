---
title: "Memory OOM"
date:   2020-08-21 00:00:01 -0800
categories:
  - blog
  - python
  - typing
authors:
  - chenwei
---

<div class="ox-hugo-toc toc">
<div></div>

<div class="heading">Table of Contents</div>

- [Example](#example)
    - [Demo program 1: allocate memory without using it.](#demo-program-1-allocate-memory-without-using-it-dot)
    - [Demo program 2: allocate memory and actually touch it all.](#demo-program-2-allocate-memory-and-actually-touch-it-all-dot)
    - [Demo program 3: first allocate, and use later.](#demo-program-3-first-allocate-and-use-later-dot)
- [Turning off overcommit](#turning-off-overcommit)
    - [Going in the wrong direction](#going-in-the-wrong-direction)
    - [Going in the right direction](#going-in-the-right-direction)
- [Resource](#resource)

</div>
<!--endtoc-->

> Lorem ipsum dolor sit amet, consectetuer adipiscing elit.  Donec hendrerit tempor tellus.  Donec pretium posuere tellus.  Proin quam nisl, tincidunt et, mattis eget, convallis nec, purus.  Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus.  Nulla posuere.  Donec vitae dolor.  Nullam tristique diam non turpis.  Cras placerat accumsan nulla.  Nullam rutrum.  Nam vestibulum accumsan nisl.

Normally, a user-space program reserves (virtual)  by calling `malloc()`. If the return value is NULL, the program knows that no more memory is available, and can do something appropriate. Most programs will print an error message and exit, some first need to clean up lockfiles or so, and some smarter programs can do garbage collection, or adapt the computation to the amount of available memory. This is life under Unix, and all is well.

Linux on the other hand is seriously broken. It will by default answer "`yes`" to most requests for memory, in the hope that programs `ask for more than they actually need`. If the hope is fulfilled Linux can run more programs in the same memory, or can run a program that requires more virtual memory than is available. And if not then very bad things happen.

| Type        | DSP Address             | ARM/DRAM physical address | Note              |
|-------------|-------------------------|---------------------------|-------------------|
| DSP Program | `0x00000000-0x0000ffff` | N/A                       | TCM(internal)     |
|             | `0x00010000-0xffffffff` | `0x00010000-0xffffffff`   | EPP(external)     |
| DSP Data    | `0x00000000-0x00007fff` | N/A                       | ISDM 64KB block 0 |
|             | `0x00008000-0x0000ffff` | N/A                       | ISDM 64KB block 1 |
|             | `0x00040000-0x00043fff` | N/A                       | IVDM 64KB block 0 |
|             | `0x00044000-0x00047fff` | N/A                       | IVDM 64KB block 1 |
|             | `0x00048000-0x0004bfff` | N/A                       | IVDM 64KB block 2 |
|             | `0x0004c000-0x0004ffff` | N/A                       | IVDM 64KB block 3 |
|             | `0x00080000-0x00087fff` | N/A                       | CIU               |
|             | `0x00100000-0xffffffff` | `0x00100000-0xffffffff`   | EDP               |

Note: DSP does not have any address translation except for IACU translation, i.e. DSP uses physical address


## Example {#example}

Typically, the first demo program will get a very large amount of memory before `malloc()` returns `NULL`.

The second demo program will get a much smaller amount of memory, now that earlier obtained memory is actually used.

The third program will get the same large amount as the first program, and then is killed when it wants to use its memory. (On a well-functioning system, like Solaris, the three demo programs obtain the same amount of memory and do not crash but see `malloc()` return `NULL`.)

For example:

-   On an 8 MiB machine without swap running 1.2.11:
    demo1: 274 MiB, demo2: 4 MiB, demo3: 270 / `oom after 1 MiB: Killed`.
-   Idem, with 32 MiB swap:
    demo1: 1528 MiB, demo2: 36 MiB, demo3: 1528 / `oom after 23 MiB: Killed`.
-   On a 32 MiB machine without swap running 2.0.34:
    demo1: 1919 MiB, demo2: 11 MiB, demo3: 1919 / `oom after 4 MiB: Bus error`.
-   Idem, with 62 MiB swap:
    demo1: 1919 MiB, demo2: 81 MiB, demo3: 1919 / oom after 74 MiB: The machine hangs. After several seconds: `Out of memory for bash. Out of memory for crond. Bus error`.
-   On a 256 MiB machine without swap running 2.6.8.1:
    demo1: 2933 MiB, demo2: after 98 MiB: Killed. Also: `Out of Memory: Killed process 17384 (java_vm)`. demo3: 2933 / `oom after 135 MiB: Killed`.
-   Idem, with 539 MiB swap:
    demo1: 2933 MiB, demo2: after 635 MiB: `Killed`. demo3: `oom after 624 MiB: Killed`.

Other kernels have other strategies. Sometimes processes get a segfault when accessing memory that the kernel is unable to provide, sometimes they are killed, sometimes other processes are killed, sometimes the kernel hangs.


### Demo program 1: allocate memory without using it. {#demo-program-1-allocate-memory-without-using-it-dot}

```c
#include <stdio.h>
#include <stdlib.h>

int main (void) {
		int n = 0;

		while (1) {
				if (malloc(1<<20) == NULL) {
						printf("malloc failure after %d MiB\n", n);
						return 0;
				}
				printf ("got %d MiB\n", ++n);
		}
}
```


### Demo program 2: allocate memory and actually touch it all. {#demo-program-2-allocate-memory-and-actually-touch-it-all-dot}

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

int main (void) {
		int n = 0;
		char *p;

		while (1) {
				if ((p = malloc(1<<20)) == NULL) {
						printf("malloc failure after %d MiB\n", n);
						return 0;
				}
				memset (p, 0, (1<<20));
				printf ("got %d MiB\n", ++n);
		}
}
```


### Demo program 3: first allocate, and use later. {#demo-program-3-first-allocate-and-use-later-dot}

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

#define N       10000

int main (void) {
		int i, n = 0;
		char *pp[N];

		for (n = 0; n < N; n++) {
				pp[n] = malloc(1<<20);
				if (pp[n] == NULL)
						break;
		}
		printf("malloc failure after %d MiB\n", n);

		for (i = 0; i < n; i++) {
				memset (pp[i], 0, (1<<20));
				printf("%d\n", i+1);
		}

		return 0;
}
```


## Turning off overcommit {#turning-off-overcommit}


### Going in the wrong direction {#going-in-the-wrong-direction}

Since 2.1.27 there are a sysctl `VM_OVERCOMMIT_MEMORY` and proc file `/proc/sys/vm/overcommit_memory` with values `1: do overcommit`, and `0 (default): don't`.

Unfortunately, this does not allow you to tell the kernel to be more careful, it only allows you to tell the kernel to be `less careful`.

With `overcommit_memory` set to 1 every malloc() will succeed.
When set to 0 the old heuristics are used, the kernel still overcommits.


### Going in the right direction {#going-in-the-right-direction}

Since 2.5.30 the values are: 0 (default): as before: guess about how much overcommitment is reasonable, `1: never refuse any malloc()`, `2: be precise about the overcommit` - never commit a virtual address space larger than swap space plus a fraction `overcommit_ratio` of the physical memory.

Here `/proc/sys/vm/overcommit_ratio` (by default 50) is another user-settable parameter. It is possible to set `overcommit_ratio` to values larger than 100. (See also `Documentation/vm/overcommit-accounting`.)

After

```text
# echo 2 > /proc/sys/vm/overcommit_memory
```

all three demo programs were able to obtain 498 MiB on this 2.6.8.1 machine (256 MiB, 539 MiB swap, lots of other active processes), very satisfactory.

However, without swap, no more processes could be started - already more than half of the memory was committed. After

```text
# echo 80 > /proc/sys/vm/overcommit_ratio
```

all three demo programs were able to obtain 34 MiB. (Exercise: solve two equations with two unknowns and conclude that main memory was 250 MiB, and the other processes took 166 MiB.)

One can view the currently committed amount of memory in /proc/meminfo, in the field `Committed_AS`.


## Resource {#resource}

-   [理解 LINUX 的 MEMORY OVERCOMMIT](http://linuxperf.com/?p=102)
-   <https://www.kernel.org/doc/Documentation/vm/overcommit-accounting>
-   <https://www.kernel.org/doc/Documentation/sysctl/vm.txt>
