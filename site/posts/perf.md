+++
title = "Perf_event"
linkTitle = "Perf event"
author = ["Chenwei Yang"]
date = 2020-11-23
lastmod = 2020-11-24
tags = ["hi", "test", "linux"]
draft = false
math = true
+++

<div class="ox-hugo-toc toc">
<div></div>

<div class="heading">Table of Contents</div>

- [Analyze uvc-gadget](#analyze-uvc-gadget)
    - [List Events](#list-events)
    - [Count Event](#count-event)
    - [Capture Stack](#capture-stack)
    - [Report](#report)
    - [Flame Graph](#flame-graph)
- [常用指令概覽](#常用指令概覽)

</div>
<!--endtoc-->



## Analyze uvc-gadget {#analyze-uvc-gadget}


### List Events {#list-events}

```text
# /home/bsp/linux/kernel/stable/tools/perf/perf list

List of pre-defined events (to be used in -e):

  branch-instructions OR branches                    [Hardware event]
  [...]

  alignment-faults                                   [Software event]
  bpf-output                                         [Software event]
  [...]

  L1-dcache-load-misses                              [Hardware cache event]
  L1-dcache-loads                                    [Hardware cache event]
  [...]

  armv7_cortex_a5/br_immed_retired/                  [Kernel PMU event]
  armv7_cortex_a5/br_mis_pred/                       [Kernel PMU event]
  armv7_cortex_a5/br_pred/                           [Kernel PMU event]
  armv7_cortex_a5/br_return_retired/                 [Kernel PMU event]
  [...]

  asoc:snd_soc_bias_level_done                       [Tracepoint event]
  asoc:snd_soc_bias_level_start                      [Tracepoint event]
  asoc:snd_soc_dapm_connected                        [Tracepoint event]
  [...]
```


### Count Event {#count-event}

```text
# /home/bsp/linux/kernel/stable/tools/perf/perf stat ./uvc-gadget -o 0 -d -f 0 -
i testYUV.yuv

 Performance counter stats for './uvc-gadget -o 0 -d -f 0 -i testYUV.yuv':

      11512.062369      task-clock (msec)         #    0.698 CPUs utilized
               170      context-switches          #    0.015 K/sec
                 0      cpu-migrations            #    0.000 K/sec
              1891      page-faults               #    0.164 K/sec
        8269998306      cycles                    #    0.718 GHz                      (74.67%)
        3856581609      instructions              #    0.47  insn per cycle           (75.32%)
         495489057      branches                  #   43.041 M/sec                    (75.03%)
          85159880      branch-misses             #   17.19% of all branches          (49.71%)

      16.481647554 seconds time elapsed
```

<!--list-separator-->

-  Syscall Counts

    ```text
    # /home/bsp/linux/kernel/stable/tools/perf/perf stat -e 'raw_syscalls:sys_enter'
     -e 'syscalls:sys_enter_*' ./uvc-gadget -o 0 -d -f 0 -i testYUV.yuv

     Performance counter stats for './uvc-gadget -o 0 -d -f 0 -i testYUV.yuv':

                844556      raw_syscalls:sys_enter
                     0      syscalls:sys_enter_socket
                     0      syscalls:sys_enter_socketpair
                     0      syscalls:sys_enter_bind
    				 [...]
                843479      syscalls:sys_enter_select
                     0      syscalls:sys_enter_pselect6
                     0      syscalls:sys_enter_poll
                     0      syscalls:sys_enter_ppoll
                   336      syscalls:sys_enter_ioctl
    				 [...]
                     2      syscalls:sys_enter_lseek
                     0      syscalls:sys_enter_llseek
                     3      syscalls:sys_enter_read
                   698      syscalls:sys_enter_write
    				 [...]

          11.145383137 seconds time elapsed
    ```


### Capture Stack {#capture-stack}

```text
# /home/bsp/linux/kernel/stable/tools/perf/perf record -g -e cycles,instructions
,L1-dcache-load-misses ./uvc-gadget -o 0 -d -f 0 -i testYUV.yuv

[ perf record: Woken up 16 times to write data ]
[ perf record: Captured and wrote 3.906 MB perf.data (37198 samples) ]
```


### Report {#report}

<!--list-separator-->

-  Caller-based Call Graph (Top Down)

    ```text
    # /home/bsp/linux/kernel/stable/tools/perf/perf report -g graph,0.5,caller
    # To display the perf.data header info, please use --header/--header-only options.
    #
    # Total Lost Samples: 0
    #
    # Samples: 13K of event 'cycles'
    # Event count (approx.): 3822458430
    #
    # Children      Self  Command     Shared Object        Symbol
    # ........  ........  ..........  ...................  ........................................
    #
        71.68%     0.00%  uvc-gadget  [unknown]            [k] 0xb6f24c38
                |
                ---0xb6f24c38
                   |
                   |--67.91%--0x1589c
                   |          |
                   |          |--59.87%--__ret_fast_syscall
                   |          |          |
                   |          |           --58.84%--sys_select
                   |          |                                |
                   |          |                                |--32.90%--do_select
                   |          |                                |          |
                   |          |                                |          |--14.95%--v4l2_poll
    			   [...]
                   |          |                                |--4.05%--arm_copy_from_user
                   |          |                                |
                   |          |                                |--3.65%--arm_copy_to_user
    			   [...]
                   |--1.41%--0x17d98
                   |          __ret_fast_syscall
                   |          sys_read
                   |          vfs_read
                   |          __vfs_read
                   |          nfs_file_read
                   |          |
                   |           --1.40%--generic_file_read_iter
    			   [...]
    ```

<!--list-separator-->

-  Callee-based Call Graph (Bottom Up)

    ```text
    # /home/bsp/linux/kernel/stable/tools/perf/perf report  --no-children -g graph,0
    .2,callee
    # To display the perf.data header info, please use --header/--header-only options.
    #
    # Total Lost Samples: 0
    #
    # Samples: 13K of event 'cycles'
    # Event count (approx.): 3822458430
    #
    # Overhead  Command     Shared Object        Symbol
    # ........  ..........  ...................  ........................................
    #
        11.48%  uvc-gadget  [kernel.kallsyms]    [k] core_sys_select
                |
                ---core_sys_select
                   |
                   |--11.12%--sys_select
                   |          __ret_fast_syscall
                   |          0x1589c
                   |          0xb6f24c38
                   |
                    --0.35%--__ret_fast_syscall
                              0x1589c
                              0xb6f24c38

         9.23%  uvc-gadget  [kernel.kallsyms]    [k] do_select
                |
                ---do_select
                   |
                   |--8.89%--core_sys_select
                   |          sys_select
                   |          __ret_fast_syscall
                   |          0x1589c
                   |          0xb6f24c38
                   |
                    --0.34%--sys_select
                              __ret_fast_syscall
                              0x1589c
                              0xb6f24c38

         5.34%  uvc-gadget  [kernel.kallsyms]    [k] arm_copy_to_user
                |
                ---arm_copy_to_user
    ```

<!--list-separator-->

-  Report perf.data on Ubuntu

    ```text
    sudo perf report -k /path/to/your/vmlinux -g graph,0.1,caller
    sudo perf report -k /path/to/your/vmlinux --no-children -g graph,0.2,callee
    ```


### Flame Graph {#flame-graph}

```text
sudo perf script -k ~/workspace/vatics/bsp/linux/kernel/stable/vmlinux -i perf.data | \
    stackcollapse-perf.pl --all | \
    flamegraph.pl --color=java --hash > flamegraph.svg
```


## 常用指令概覽 {#常用指令概覽}

perf 指令有很多子指令（Sub-command）。本文只會涵蓋以下 4 個重要指令：

perf list
: 列出 Linux 核心與硬體支援的事件種類。

perf stat [-e [events]] [cmd]
: 執行指定程式。執行結束後，印出各 事件的總體統計數據。
    -   選項 `-e` 用以指定要測量的 事件種類。各事件種類以 `,` 分隔。

perf record [-e [events]] [-g] [--call graph [fp,dwarf,lbr]] [cmd]
: 執行指定程式。執行過程中，取樣並記 錄各事件。 這個指令能讓我們能大略 知道事件的發生地點。
    -   選項 `-e` 用以指定要測量 的事件種類。各事件種類以 `,` 分隔。
    -   選項 `-g` 讓 `perf record` 在記錄事 件時，同時記錄取樣點的 Stack Trace
    -   可以透過 `--call-graph` 選項指定走訪 Stack Trace 的方法
    -   選項 `-o` 用以指定一個檔 案名稱作為事件記錄儲存位置。預設值為 `perf.data`

perf report [-g graph,0.5,caller]
: 讀取 perf record 的記錄。
    -   選項 `-g` 用以指定樹狀圖 的繪製方式。 `graph,0.5,caller` 代表以 呼叫者（Caller）作為父節點，以 被呼叫者（Callee）作為子節點。 這是最新版 perf 指令的 預設值。
    -   選項 `-i` 用以指定一個檔 案名稱作為事件記錄讀取來源。預 設值為 perf.data。

[Note] 若測量對象沒有安全疑慮，以上所有指令都建議使用 `sudo` 執行。
