# 12. Lab: Forensic analysis of computer memory

## Instructions

1. While playing a video file, insert another video file to play.

## More information

## Detailed instructions

### 1. Forensic analysis of computer memory

In general, a computer according to [von Neumann architecture](https://en.wikipedia.org/wiki/Von_Neumann_architecture) consists of a central processing unit, memory and input/output devices.

We usually perform forensic analysis on disks, but in the case of encrypted disks, we cannot access the data if we do not know the key to access the data. In order to analyze an encrypted disk, we must first analyze the memory and find the encryption key in it. The memory also contains passwords, temporary files and other temporary settings.

In order to get to the data in the memory of a well-protected, locked and encrypted computer, we need to perform a [Cold Boot attack](https://en.wikipedia.org/wiki/Cold_boot_attack).

Every computer has memory in which it stores temporary files, files that are currently in use and environmental variables, as well as operating system and program settings. Memory consists of cells that contain capacitors that hold a charge that can be used to store data. The data is stored in the capacitor when we have power and the charge in the capacitor can be refreshed frequently. When we turn off the power supply, the recharging or the data stops, but the charge remains in the capacitors for some time. How long depends on the construction of the capacitor, the properties of the materials and the temperature. The lower the temperature, the longer the charge remains in the capacitor and the higher the temperature, the faster the charge is lost.

The goal of a Cold Boot attack is to try to cool down the memory as much as possible with [liquid nitrogen](https://en.wikipedia.org/wiki/Liquid_nitrogen) or [freeze spray](https://en.wikipedia.org/wiki/Freeze_spray) to the lowest possible temperature. Once the memory has cooled, we quickly transfer it to another working computer and read it. In this way, we can make an image of the memory, which can then be used for further analysis.

We need a customized computer that allows connecting and reading the memory during operation, since computers usually erase their memory when they are turned on and/or turned off. This property depends on how [BIOS](https://en.wikipedia.org/wiki/BIOS)/[UEFI](https://en.wikipedia.org/wiki/UEFI) initializes memory.

To read specific memory, we need a processor controller and motherboard dedicated to the correct type of memory. Today, the most common type of memory is the [Double Data Rate (DDR)](Double_data_rate) version [3](https://en.wikipedia.org/wiki/DDR3_SDRAM), [4](https://en.wikipedia.org/wiki/DDR4_SDRAM) and [5](https://en.wikipedia.org/wiki/DDR5_SDRAM).

The computer starts up by receiving power, which triggers the [program counter](https://en.wikipedia.org/wiki/Program_counter) to be set to its default value, through which the BIOS/UEFI boots, which take care of verifying and starting the hardware. It then hands off execution to the [bootloader](https://en.wikipedia.org/wiki/Bootloader), which takes care of booting the operating system.

In an ideal world, you would write your own BIOS/UEFI that would not destroy the data in the memory and allow reading without initialization. However, BIOS/UEFI is usually a closed software that is written on a separate memory on the motherboard and is difficult to change. But we can write any bootloader that would allow us to do this, since it is just a piece of any software located on any disk. We don't want to read the memory anymore while the operating system is booting and running, because it is the only one using the memory and has already allocated a lot of important locations. Examples of modified bootloaders are:
- [msramdump](https://github.com/Schramp/msramdump)
- [memimage](https://citp.princeton.edu/our-work/memory/code)

In order to defend against a Cold Boot attack, we should not have confidential data stored in memory, such as an encryption key. The processor has specially separated registers and a small memory that is difficult to access, does not lose data during power loss and is intended to store confidential data. Separate memory is managed by a dedicated separate processor with dedicated software and communicates with the main processor via an agreed upon protocol. This functionality is known on Intel processors under the name [Intel Management Engine (IME)](https://en.wikipedia.org/wiki/Intel_Management_Engine) and on AMD processors under the name [Platform Security Processor (PSP)](https://en.wikipedia.org/wiki/AMD_Platform_Security_Processor) or [Microsoft Pluto](https://www.microsoft.com/en-us/security/blog/2020/11/17/meet-the-microsoft-pluton-processor-the-security-chip-designed-for-the-future-of-windows-pcs/). This begs the question, if do we trust a closed system to look after our confidential data? Similar functionality is also performed by [smart cards](https://en.wikipedia.org/wiki/Smart_card), which are connected to a computer via an input/output controller and contain their own separate processor that performs a similar functionality to an IME and PSP.

We have a large amount of memory on our computer and now we will see how we can access individual parts of it. Modern operating systems use virtual memory, which resides in memory and also extends to disk, and is defined by a [page table](https://en.wikipedia.org/wiki/Page_table). We can make a copy of the entire memory, where the operating system, all programs, current files, settings and environment are located, and then analyze it. But we can focus on the memory of an individual program and find out what it has in memory, which files it has open, how it is executed, and the like.

We have dedicated tools for analyzing program memory:
- Windows: [mimiKatz](https://github.com/ParrotSec/mimikatz)
- Linux: [volatility](https://github.com/volatilityfoundation/volatility), [rekall](https://github.com/google/rekall), [yara](https://github.com/VirusTotal/yara)

But we can try to analyze and change the program memory ourselves. Let's install a program for downloading videos from the web [`yt-dlp`](https://github.com/yt-dlp/yt-dlp) and use it to download two videos from the web and save them with the same format. We play them with the program [`vlc`](https://www.videolan.org/vlc/).

    apt update
    apt instal curl ffmpeg vlc

    sudo curl -L https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp -o /usr/local/bin/yt-dlp
    sudo chmod a+rx /usr/local/bin/yt-dlp

    yt-dlp -f "248+251" https://www.youtube.com/watch?v=2uzlfzrMu0E
    yt-dlp -f "248+251" https://www.youtube.com/watch?v=ag2O2z99shY

    ls

    'Why Norway Is One Of The BEST Destinations In The World [2uzlfzrMu0E].webm'
    'Why Slovenia Is The BEST Vacation Spot You'\''ve Never Heard Of [ag2O2z99shY].webm'

    mv Why* /home/aleks

    mv 'Why Norway Is One Of The BEST Destinations In The World [2uzlfzrMu0E].webm' Norway.webm

    mv 'Why Slovenia Is The BEST Vacation Spot You'\''ve Never Heard Of [ag2O2z99shY].webm' Slovenia.webm

    vlc Slovenia.webm

With the command [`ps`](https://www.man7.org/linux/man-pages/man1/ps.1.html) we can list all the processes currently running in our operating system. The label `PID - Process Identifier` identifies each process and allows us to manage it.

    ps -a

         PID TTY          TIME CMD
        1061 tty2     00:00:00 gnome-session-b
        3305 pts/1    00:00:00 su
        3306 pts/1    00:00:00 bash
        3404 pts/0    00:00:06 vlc
        3469 pts/1    00:00:00 ps


Linux has a virtual directory `/dev` which addresses all devices, `/sys` which addresses modules, subsystems and non-process settings and [`/proc`](https://man7.org/linux/man-pages/man5/proc.5.html) which addresses all currently running processes.

    ls /proc

    1     1154  1277  1452	2195  3471  58	       dma	      pagetypeinfo
    10    1161  1278  1454	22    382   59	       driver	      partitions
    1000  1167  128   1455	2218  386   590        dynamic_debug  pressure
    1001  117   1281  1456	23    388   593        execdomains    sched_debug
    1015  119   129   1466	2314  390   594        fb	      schedstat
    1016  1191  1291  1469	24    393   6	       filesystems    self
    1018  12    1293  1479	241   399   640        fs	      slabinfo
    1023  120   1294  1486	25    4     68	       interrupts     softirqs
    1028  121   1296  1487	26    400   71	       iomem	      stat
    1043  1215  13	  1491	27    401   72	       ioports	      swaps
    1045  1219  130   1496	28    402   728        irq	      sys
    1055  122   1302  15	288   405   799        kallsyms       sysrq-trigger
    1058  1226  1303  1521	29    419   8	       kcore	      sysvipc
    1061  123   131   16	296   430   9	       keys	      thread-self
    1069  1234  132   17	3     473   928        key-users      timer_list
    1078  124   1321  177	30    475   995        kmsg	      tty
    109   1244  133   178	31    477   acpi       kpagecgroup    uptime
    1090  1245  1331  18	3223  486   asound     kpagecount     version
    1095  1257  134   1968	3303  489   buddyinfo  kpageflags     vmallocinfo
    11    1258  135   1970	3305  49    bus        loadavg	      vmstat
    1102  1268  136   1973	3306  50    cgroups    locks	      zoneinfo
    1109  1269  137   2	3309  51    cmdline    meminfo
    1115  1270  1371  20	3320  52    consoles   misc
    112   1271  1375  2003	3325  53    cpuinfo    modules
    1136  1272  1378  2096	3326  55    crypto     mounts
    1137  1273  1439  2098	3402  56    devices    mtrr
    1139  1276  1443  215	3404  57    diskstats  net

    ls /proc/3404

    arch_status	    cwd        mem	      patch_state   stat
    attr		    environ    mountinfo      personality   statm
    autogroup	    exe        mounts	      projid_map    status
    auxv		    fd	       mountstats     root	    syscall
    cgroup		    fdinfo     net	      sched	    task
    clear_refs	    gid_map    ns	      schedstat     timens_offsets
    cmdline		    io	       numa_maps      sessionid     timers
    comm		    limits     oom_adj	      setgroups     timerslack_ns
    coredump_filter     loginuid   oom_score      smaps	    uid_map
    cpu_resctrl_groups  map_files  oom_score_adj  smaps_rollup  wchan
    cpuset		    maps       pagemap	      stack

The `mem` file addresses the entire virtual memory of the process as seen by the process itself with relative offsets. When we try to save it, it returns an error because it tries to read parts of the memory that don't exist.

    cat /proc/3404/mem > /home/aleks/vlcmemdump.raw

    cat: /proc/3404/mem: Input/output error

The `maps` file tells us which parts of the virtual memory in the `mem` file are used by the process and allows us to further analyze the process memory. The first two numbers represent the part of the memory that the line addresses (the address of the first byte and the address of the first byte after the last byte of the memory part in hexadecimal), followed by permissions and information about the file (offset, device, inode and name). Addresses that point to the `/usr/bin/...` folder tell you which program is running. Memory starts with the program code, constants, local and dynamic variables, then comes free memory, and finally we have the stack and return address.

    cat /proc/3404/maps > /home/aleks/vlcmapsdump
    less /home/aleks/vlcmapsdump

    55674e210000-55674e211000 r--p 00000000 08:01 976854                     /usr/bin/vlc
    55674e211000-55674e212000 r-xp 00001000 08:01 976854                     /usr/bin/vlc
    55674e212000-55674e213000 r--p 00002000 08:01 976854                     /usr/bin/vlc
    55674e213000-55674e214000 r--p 00002000 08:01 976854                     /usr/bin/vlc
    55674e214000-55674e215000 rw-p 00003000 08:01 976854                     /usr/bin/vlc
    55674fe66000-55674ff8f000 rw-p 00000000 00:00 0                          [heap]
    7f3a78000000-7f3a7bb2c000 rw-p 00000000 00:00 0 
    7f3a7bb2c000-7f3a7c000000 ---p 00000000 00:00 0 
    7f3a7cf94000-7f3a7d994000 rwxp 00000000 00:00 0 
    7f3a7d994000-7f3a7d995000 ---p 00000000 00:00 0 
    7f3a7d995000-7f3a7e195000 rw-p 00000000 00:00 0 
    7f3a7e195000-7f3a7e197000 r--p 00000000 08:01 1102753                    /usr/lib/x86_64-linux-gnu/vlc/pl
    ugins/codec/libvaapi_plugin.so
    7f3a7e197000-7f3a7e19a000 r-xp 00002000 08:01 1102753                    /usr/lib/x86_64-linux-gnu/vlc/pl
    ugins/codec/libvaapi_plugin.so
    7f3a7e19a000-7f3a7e19b000 r--p 00005000 08:01 1102753                    /usr/lib/x86_64-linux-gnu/vlc/pl
    ugins/codec/libvaapi_plugin.so
    7f3a7e19b000-7f3a7e19c000 ---p 00006000 08:01 1102753                    /usr/lib/x86_64-linux-gnu/vlc/pl
    ugins/codec/libvaapi_plugin.so
    7f3a7e19c000-7f3a7e19d000 r--p 00006000 08:01 1102753                    /usr/lib/x86_64-linux-gnu/vlc/pl
    ugins/codec/libvaapi_plugin.so
    7f3a7e19d000-7f3a7e19e000 rw-p 00007000 08:01 1102753                    /usr/lib/x86_64-linux-gnu/vlc/pl
    ugins/codec/libvaapi_plugin.so
    7f3a7e19e000-7f3a7e1a0000 r--p 00000000 08:01 1103016                    /usr/lib/x86_64-linux-gnu/vlc/pl
    ugins/vaapi/libvaapi_filters_plugin.so
    7f3a7e1a0000-7f3a7e1b8000 r-xp 00002000 08:01 1103016                    /usr/lib/x86_64-linux-gnu/vlc/pl
    ugins/vaapi/libvaapi_filters_plugin.so
    7f3a7e1b8000-7f3a7e1bb000 r--p 0001a000 08:01 1103016                    /usr/lib/x86_64-linux-gnu/vlc/pl
    ugins/vaapi/libvaapi_filters_plugin.so
    7f3a7e1bb000-7f3a7e1bc000 r--p 0001c000 08:01 1103016                    /usr/lib/x86_64-linux-gnu/vlc/pl
    ugins/vaapi/libvaapi_filters_plugin.so
    7f3a7e1bc000-7f3a7e1bd000 rw-p 0001d000 08:01 1103016                    /usr/lib/x86_64-linux-gnu/vlc/pl
    ugins/vaapi/libvaapi_filters_plugin.so
    7f3a7e1bd000-7f3a7e1bf000 r--p 00000000 08:01 1103030                    /usr/lib/x86_64-linux-gnu/vlc/pl
    ugins/video_output/libglconv_vaapi_drm_plugin.so
    7f3a7e1bf000-7f3a7e1c2000 r-xp 00002000 08:01 1103030                    /usr/lib/x86_64-linux-gnu/vlc/pl
    ugins/video_output/libglconv_vaapi_drm_plugin.so
    7f3a7e1c2000-7f3a7e1c4000 r--p 00005000 08:01 1103030                    /usr/lib/x86_64-linux-gnu/vlc/pl
    ugins/video_output/libglconv_vaapi_drm_plugin.so
    7f3a7e1c4000-7f3a7e1c5000 r--p 00006000 08:01 1103030                    /usr/lib/x86_64-linux-gnu/vlc/pl
    ugins/video_output/libglconv_vaapi_drm_plugin.so
    7f3a7e1c5000-7f3a7e1c6000 rw-p 00007000 08:01 1103030                    /usr/lib/x86_64-linux-gnu/vlc/pl
    ugins/video_output/libglconv_vaapi_drm_plugin.so
    7f3a7e1c6000-7f3a7e1c8000 r--p 00000000 08:01 1103013                    /usr/lib/x86_64-linux-gnu/vlc/li
    bvlc_vdpau.so.0.0.0

Now we can write a short program that prints the entire memory of the process according to the contents of the `mem` and `maps` files, or we can download the [program](https://davidebove.com/blog/how-to-dump-process-memory-in-linux/) from the web and run it for the desired `PID`. The result is a binary file that represents the memory of the process and is intended for further analysis.

    wget https://gist.githubusercontent.com/Dbof/b9244cfc607cf2d33438826bee6f5056/raw/aa4b75ddb55a58e2007bf12e17daadb0ebebecba/memdump.py

    sudo python3 memdump.py 3404

The `fd` directory contains symbolic links, named with numbers, to the open files of this process. Symbolic links allow us to open a file and read it, even if the file was deleted from the disk in the meantime. To find out where the symbolic links point to, use the `ls -al` command and see that the number 32 points to the video that is currently playing.

    ls /proc/3404/fd

    0  10  12  14  16  18  2   21  23  25  29  30  32  34  4  6  8
    1  11  13  15  17  19  20  22  24  26  3   31  33  35  5  7  9

    ls -al /proc/3404/fd

    total 0
    dr-x------ 2 aleks aleks  0 May 16 14:30 .
    dr-xr-xr-x 9 aleks aleks  0 May 16 14:30 ..
    lrwx------ 1 aleks aleks 64 May 16 14:30 0 -> /dev/pts/1
    lrwx------ 1 aleks aleks 64 May 16 14:30 1 -> /dev/pts/1
    lrwx------ 1 aleks aleks 64 May 16 14:30 10 -> 'socket:[57854]'
    lrwx------ 1 aleks aleks 64 May 16 14:30 11 -> 'socket:[58753]'
    lrwx------ 1 aleks aleks 64 May 16 14:30 12 -> 'anon_inode:[eventfd]'
    lrwx------ 1 aleks aleks 64 May 16 14:30 13 -> 'anon_inode:[eventfd]'
    lrwx------ 1 aleks aleks 64 May 16 14:30 14 -> 'socket:[58755]'
    lrwx------ 1 aleks aleks 64 May 16 14:30 15 -> 'anon_inode:[eventfd]'
    lrwx------ 1 aleks aleks 64 May 16 14:30 16 -> 'anon_inode:[eventfd]'
    lrwx------ 1 aleks aleks 64 May 16 14:30 17 -> '/memfd:wayland-cursor (deleted)'
    lrwx------ 1 aleks aleks 64 May 16 14:30 18 -> 'socket:[58764]'
    lrwx------ 1 aleks aleks 64 May 16 14:30 19 -> 'socket:[58766]'
    lrwx------ 1 aleks aleks 64 May 16 14:30 2 -> /dev/pts/1
    lrwx------ 1 aleks aleks 64 May 16 14:30 20 -> 'anon_inode:[eventfd]'
    lrwx------ 1 aleks aleks 64 May 16 14:30 21 -> 'socket:[58756]'
    lrwx------ 1 aleks aleks 64 May 16 14:30 22 -> 'anon_inode:[eventfd]'
    lr-x------ 1 aleks aleks 64 May 16 14:30 23 -> anon_inode:inotify
    lrwx------ 1 aleks aleks 64 May 16 14:30 24 -> 'anon_inode:[eventfd]'
    lrwx------ 1 aleks aleks 64 May 16 14:30 25 -> 'socket:[57866]'
    lrwx------ 1 aleks aleks 64 May 16 14:30 26 -> 'socket:[58768]'
    lr-x------ 1 aleks aleks 64 May 16 14:30 29 -> /usr/share/icons/Adwaita/icon-theme.cache
    lr-x------ 1 aleks aleks 64 May 16 14:30 3 -> 'pipe:[57849]'
    lr-x------ 1 aleks aleks 64 May 16 14:30 30 -> /usr/share/icons/hicolor/icon-theme.cache
    lrwx------ 1 aleks aleks 64 May 16 14:30 31 -> 'socket:[57886]'
    lr-x------ 1 aleks aleks 64 May 16 14:30 32 -> '/home/aleks/Slovenia.webm'
    lrwx------ 1 aleks aleks 64 May 16 14:30 33 -> 'socket:[58797]'
    lrwx------ 1 aleks aleks 64 May 16 14:30 34 -> 'socket:[58799]'
    lrwx------ 1 aleks aleks 64 May 16 14:30 35 -> 'socket:[57948]'
    l-wx------ 1 aleks aleks 64 May 16 14:30 4 -> 'pipe:[57849]'
    lrwx------ 1 aleks aleks 64 May 16 14:30 5 -> 'anon_inode:[eventfd]'
    lrwx------ 1 aleks aleks 64 May 16 14:30 6 -> 'socket:[57851]'
    lrwx------ 1 aleks aleks 64 May 16 14:30 7 -> 'anon_inode:[eventfd]'
    lr-x------ 1 aleks aleks 64 May 16 14:30 8 -> 'pipe:[57853]'
    l-wx------ 1 aleks aleks 64 May 16 14:30 9 -> 'pipe:[57853]'

The memory of the program can also be examined with the program [`radare2`](https://github.com/radareorg/radare2). With the setting `v` we enable the graphic interface and then by clicking on the menu `View\Hexdump`, where we can inspect the contents of the memory.

    apt update
    apt install git
    git clone https://github.com/radareorg/radare2
    radare2/sys/install.sh

    r2 3404.dump

    v

    View\Hexdump

    q

    q

For example, we can also examine the memory of the `nano` text editor. By opening it and writing any text in it and leaving it open. Then we check the editor's `PID` with the `ps` command and make a copy of the memory with the `memdump.py` script. The contents of the memory can then be written out with the command [`strings`](https://www.man7.org/linux/man-pages/man1/strings.1.html) and then filtered with the command [`grep`](https://man7.org/linux/man-pages/man1/grep.1.html) by a keyword, for example `test`. Other more advanced programs can also extract the data structures used by the process from the memory copy.

    nano test

    This is a test string!

    ps -a

     PID TTY          TIME CMD
    1095 tty2     00:00:00 gnome-session-b
    1720 pts/0    00:00:00 su
    1727 pts/0    00:00:00 bash
    43926 pts/1    01:17:33 vlc
    55197 pts/2    00:00:00 su
    55198 pts/2    00:00:00 bash
    56021 pts/2    00:00:00 nano
    56032 pts/0    00:00:00 ps

    sudo python3 memdump.py 56021

    strings 56021.dump | grep test

    quotestr
    --quotestr=<regex>
    test
    This is a test string!
    ./.test.swp
    test
    This is a test string!
    test
    glibc.malloc.arena_test
    file you run.  This is mostly of use for maintainers to test new versions
    is is a test 
    test

The [`gdb`](https://man7.org/linux/man-pages/man1/gdb.1.html) debugger allows us to attach to the currently running process. When we connect to the process, it just stops and waits to run the next command. The further execution of the process can be initiated with the command `continue` and then with the command `CTRL+C` the execution can be stopped again. The command `where` tells us where we are in memory. With the `disassemble` command, we can look at the program code of individual functions. With the `frame` command, we can print values in memory, such as the heap, the stack, and other variables and data structures.

    apt update
    apt install gdb

    gdb attach 56021

    GNU gdb (Debian 10.1-1.7) 10.1.90.20210103-git
    Copyright (C) 2021 Free Software Foundation, Inc.
    License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.
    Type "show copying" and "show warranty" for details.
    This GDB was configured as "x86_64-linux-gnu".
    Type "show configuration" for configuration details.
    For bug reporting instructions, please see:
    <https://www.gnu.org/software/gdb/bugs/>.
    Find the GDB manual and other documentation resources online at:
        <http://www.gnu.org/software/gdb/documentation/>.

    For help, type "help".
    Type "apropos word" to search for commands related to "word"...
    attach: No such file or directory.
    Attaching to process 56021
    Reading symbols from /usr/bin/nano...
    (No debugging symbols found in /usr/bin/nano)
    Reading symbols from /lib/x86_64-linux-gnu/libncursesw.so.6...
    (No debugging symbols found in /lib/x86_64-linux-gnu/libncursesw.so.6)
    Reading symbols from /lib/x86_64-linux-gnu/libtinfo.so.6...
    (No debugging symbols found in /lib/x86_64-linux-gnu/libtinfo.so.6)
    Reading symbols from /lib/x86_64-linux-gnu/libc.so.6...
    Reading symbols from /usr/lib/debug/.build-id/e1/5ec78d51a522023f9cfc58dc284f379d81860b.debug...
    Reading symbols from /lib/x86_64-linux-gnu/libdl.so.2...
    Reading symbols from /usr/lib/debug/.build-id/46/b3bf3f9b9eb092a5c0cf5575e89092f768054c.debug...
    Reading symbols from /lib64/ld-linux-x86-64.so.2...
    Reading symbols from /usr/lib/debug/.build-id/e2/5570740d590e5cb7b1a20d86332a8d1bb3b65f.debug...
    Reading symbols from /lib/x86_64-linux-gnu/libnss_files.so.2...
    Reading symbols from /usr/lib/debug/.build-id/ba/b4b71665bcc7f3f9b142804534c6de15b6e824.debug...
    0x00007fe8067663ce in __GI___libc_read (fd=0, buf=0x7ffc5b984baf, nbytes=1)
        at ../sysdeps/unix/sysv/linux/read.c:26
    26	../sysdeps/unix/sysv/linux/read.c: No such file or directory.

    (gdb) continue
    
    Continuing.
    
    ^C
    
    Program received signal SIGINT, Interrupt.
    0x00007fe8067663ce in __GI___libc_read (fd=0, buf=0x7ffc5b984baf, nbytes=1)
        at ../sysdeps/unix/sysv/linux/read.c:26
    26	in ../sysdeps/unix/sysv/linux/read.c

    (gdb) where
    
    #0  0x00007fe8067663ce in __GI___libc_read (fd=0, buf=0x7ffc5b984baf, nbytes=1)
        at ../sysdeps/unix/sysv/linux/read.c:26
    #1  0x00007fe8068906e1 in ?? () from /lib/x86_64-linux-gnu/libncursesw.so.6
    #2  0x00007fe806891277 in wgetch () from /lib/x86_64-linux-gnu/libncursesw.so.6
    #3  0x000055de729dc511 in ?? ()
    #4  0x000055de729dc845 in ?? ()
    #5  0x000055de729dd2e9 in ?? ()
    #6  0x000055de729e26e8 in ?? ()
    #7  0x000055de729ccf94 in ?? ()
    #8  0x000055de729b58e5 in ?? ()
    #9  0x00007fe80669ed0a in __libc_start_main (main=0x55de729b4ba0, argc=2, argv=0x7ffc5b9855e8, 
        init=<optimized out>, fini=<optimized out>, rtld_fini=<optimized out>, stack_end=0x7ffc5b9855d8)
        at ../csu/libc-start.c:308
    #10 0x000055de729b5faa in ?? ()

    (gdb) disassemble 0x00007fe8067663ce
    
    Dump of assembler code for function __GI___libc_read:
    0x00007fe8067663c0 <+0>:	mov    %fs:0x18,%eax
    0x00007fe8067663c8 <+8>:	test   %eax,%eax
    0x00007fe8067663ca <+10>:	jne    0x7fe8067663e0 <__GI___libc_read+32>
    0x00007fe8067663cc <+12>:	syscall 
    => 0x00007fe8067663ce <+14>:	cmp    $0xfffffffffffff000,%rax
    0x00007fe8067663d4 <+20>:	ja     0x7fe806766430 <__GI___libc_read+112>
    0x00007fe8067663d6 <+22>:	ret    
    0x00007fe8067663d7 <+23>:	nopw   0x0(%rax,%rax,1)
    0x00007fe8067663e0 <+32>:	sub    $0x28,%rsp
    0x00007fe8067663e4 <+36>:	mov    %rdx,0x18(%rsp)
    0x00007fe8067663e9 <+41>:	mov    %rsi,0x10(%rsp)
    0x00007fe8067663ee <+46>:	mov    %edi,0x8(%rsp)
    0x00007fe8067663f2 <+50>:	call   0x7fe8066fb9c0 <__libc_enable_asynccancel>
    0x00007fe8067663f7 <+55>:	mov    0x18(%rsp),%rdx
    0x00007fe8067663fc <+60>:	mov    0x10(%rsp),%rsi
    0x00007fe806766401 <+65>:	mov    %eax,%r8d
    0x00007fe806766404 <+68>:	mov    0x8(%rsp),%edi
    0x00007fe806766408 <+72>:	xor    %eax,%eax
    0x00007fe80676640a <+74>:	syscall 
    0x00007fe80676640c <+76>:	cmp    $0xfffffffffffff000,%rax
    0x00007fe806766412 <+82>:	ja     0x7fe806766448 <__GI___libc_read+136>
    0x00007fe806766414 <+84>:	mov    %r8d,%edi
    0x00007fe806766417 <+87>:	mov    %rax,0x8(%rsp)
    0x00007fe80676641c <+92>:	call   0x7fe8066fba20 <__libc_disable_asynccancel>
    0x00007fe806766421 <+97>:	mov    0x8(%rsp),%rax
    0x00007fe806766426 <+102>:	add    $0x28,%rsp
    0x00007fe80676642a <+106>:	ret    
    0x00007fe80676642b <+107>:	nopl   0x0(%rax,%rax,1)
    0x00007fe806766430 <+112>:	mov    0xe2a39(%rip),%rdx        # 0x7fe806848e70
    0x00007fe806766437 <+119>:	neg    %eax
    0x00007fe806766439 <+121>:	mov    %eax,%fs:(%rdx)
    0x00007fe80676643c <+124>:	mov    $0xffffffffffffffff,%rax
    0x00007fe806766443 <+131>:	ret    
    0x00007fe806766444 <+132>:	nopl   0x0(%rax)
    0x00007fe806766448 <+136>:	mov    0xe2a21(%rip),%rdx        # 0x7fe806848e70
    0x00007fe80676644f <+143>:	neg    %eax
    0x00007fe806766451 <+145>:	mov    %eax,%fs:(%rdx)
    0x00007fe806766454 <+148>:	mov    $0xffffffffffffffff,%rax
    0x00007fe80676645b <+155>:	jmp    0x7fe806766414 <__GI___libc_read+84>
    End of assembler dump.

    (gdb) frame
    
    #0  0x00007fe8067663ce in __GI___libc_read (fd=0, buf=0x7ffc5b984baf, nbytes=1)
        at ../sysdeps/unix/sysv/linux/read.c:26
    26	in ../sysdeps/unix/sysv/linux/read.c

Now let's try to write a program that connects to the video playback process via the `gdb` debugger. It then looks for a file descriptor to the currently playing video file. It closes the current video file and opens a new video file under its file descriptor, causing the player to start playing a new video clip. Problems arise when we use different formats for video files and in this case the program reports an error. Our program can be summarized by the following [example]((https://sajjanrajjain.wordpress.com/2015/08/29/changing-file-descriptors-in-gdb/).).

We use the command [`chmod`](https://man7.org/linux/man-pages/man1/chmod.1.html) to change the access rights of our two video files so that they can be read by any program. Next, we create our program that will receive two arguments. The first argument `NAMEKEY` represents the full or partial name of the video file to be played. The second argument `NEWFILE` represents the path and name of the file we want to play. In the next step, we get the `PID` of the video player and then get the `FD` file descriptor to the currently playing video file. To change the video file in the video player process, we use the `gdb` tool. First, we attach to the process with the `attach` command. Within the process, we now open a new video file with the system call [`open`](https://www.man7.org/linux/man-pages/man2/open.2.html), which returns file descriptor under which an open file is now available. Using the system call [`dup2`](https://man7.org/linux/man-pages/man2/dup.2.html) we clone the file descriptor from the currently playing file to a new file that we just opened it. Finally, disconnect from the process with the `detach` command and then close the `gdb` tool with the `quit` command.

Let's also change the rights of our program so that they now allow execution. We start the video player with the first video clip and then run our program with the correct parameters.    

    chmod 666 Norway.webm
    chmod 666 Slovenia.webm

    nano switcher

    #!/bin/bash

    NAMEKEY=$1
    NEWFILE=$2

    echo $NAMEKEY
    echo $NEWFILE

    PID=$(ps xa | grep "vlc" | head -n 1 | awk '{ print $1 }' )
    echo $PID

    FD=$(ls -al /proc/$PID/fd | grep $NAMEKEY | cut -d " " -f 10 )
    echo $FD

    echo -e "attach $PID\ncall open(\"$NEWFILE\", 0666)\ncall (int)dup2(\$1, $FD)\ndetach\nquit" | gdb

    chmod +x switcher

    vlc Slovenia.webm

    ./switcher Slovenia /home/aleks/Norway.webm
