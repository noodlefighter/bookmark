# Linux 修改 ELF 解决 glibc 兼容性问题

```
转自：Soul Of Free Loop
链接：https://zohead.com/archives/mod-elf-glibc/
```

## **Linux glibc 问题**

相信有不少 Linux 用户都碰到过运行第三方（非系统自带软件源）发布的程序时的 glibc 兼容性问题，这一般是由于当前 Linux 系统上的 GNU C 库（glibc）版本比较老导致的，例如我在 CentOS 6 64 位系统上运行某第三方闭源软件时会报：

```
[root@centos6-dev ~]# ldd tester
./tester: /lib64/libc.so.6: version `GLIBC_2.17' not found (required by ./tester)
./tester: /lib64/libc.so.6: version `GLIBC_2.14' not found (required by ./tester)
        linux-vdso.so.1 =>  (0x00007ffe795fe000)
        libpthread.so.0 => /lib64/libpthread.so.0 (0x00007fc7d4c73000)
        libOpenCL.so.1 => /usr/lib64/libOpenCL.so.1 (0x00007fc7d4a55000)
        libdl.so.2 => /lib64/libdl.so.2 (0x00007fc7d4851000)
        libm.so.6 => /lib64/libm.so.6 (0x00007fc7d45cd000)
        libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x00007fc7d43b7000)
        libc.so.6 => /lib64/libc.so.6 (0x00007fc7d4023000)
        /lib64/ld-linux-x86-64.so.2 (0x00007fc7d4e90000)
```

CentOS 6 自带的 glibc 还是很老的 2.12 版本，而下载的第三方程序依赖 glibc 2.17 版本，这种情况要么自己重新编译程序，要么只能升级系统的 glibc 版本。由于我使用的程序是第三方编写并且是闭源软件无法自己编译，升级 glibc 固然可能能解决问题，但是 glibc 做为最核心的基础库，在生产环境上直接升级毕竟动作还是太大，因此希望还是能有更好的解决途径。

## **问题分析**

首先我们可以检查一下程序使用了新版本 glibc 的哪些符号，使用 objdump 命令可以查看 ELF 文件的动态符号信息：

```
[root@centos6-dev ~]# objdump -T tester | grep GLIBC_2.1.*
0000000000000000      DF *UND*  0000000000000000  GLIBC_2.14  memcpy
0000000000000000      DF *UND*  0000000000000000  GLIBC_2.17  clock_gettime
```

从上面的输出可以看到程序使用了 glibc 2.14 版本的 memcpy 函数和 glibc 2.17 版本的 clock_gettime 函数，而这两个常用的函数按说应该是 glibc 很早就已经支持了的，我们可以确认一下当前系统 glibc 提供的符号版本：

```

[root@centos6-dev ~]# objdump -T /lib64/libc.so.6 | grep memcpy
0000000000091300  w   DF .text  0000000000000009  GLIBC_2.2.5 wmemcpy
0000000000101070 g    DF .text  000000000000001b  GLIBC_2.4   __wmemcpy_chk
00000000000896b0 g    DF .text  0000000000000465  GLIBC_2.2.5 memcpy
00000000000896a0 g    DF .text  0000000000000009  GLIBC_2.3.4 __memcpy_chk
[root@centos6-dev ~]# objdump -T /lib64/libc.so.6 | grep clock_gettime
000000000038f800 g    DO .bss   0000000000000008  GLIBC_PRIVATE __vdso_clock_gettime
```

这里可以看出 CentOS 6 的 glibc 库提供的 memcpy 实现是 2.2.5 版本的，另外 libc 没有直接实现 clock_gettime 函数，因为老版本 glibc 里 clock_gettime 是由 librt 库提供 clock_gettime 支持的，而且同样也是 2.2.5 版本：

```

[root@centos6-dev ~]# objdump -T /lib64/librt.so.1 | grep clock_gettime
0000000000000000      DO *UND*  0000000000000000  GLIBC_PRIVATE __vdso_clock_gettime
0000000000003e70 g    DF .text  000000000000008b  GLIBC_2.2.5 clock_gettime
```

看过这里就基本明白了，第三方程序的开发者是在自带新版本 glibc 的 Linux 系统上编译的，memcpy 和 clock_gettime 的实现默认使用了该系统上 glibc 所提供的最新版本，这样在低版本 glibc 系统中就无法正常运行。

## **解决方法**

虽然我们无法重新编译第三方程序，但如果可以修改 ELF 文件强制让 LD 库加载程序时使用老版本的 memcpy 和 clock_gettime 实现，应该就可以避免升级 glibc。

## **分析 ELF**

首先用 readelf 命令查看 ELF 的符号表，由于该命令输出非常多，这里只贴出我们关心的信息：

```

[root@centos6-dev ~]# readelf -sV tester
Symbol table '.dynsym' contains 4583 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND
    ......
    11: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND memcpy@GLIBC_2.14 (5)
    ......
    67: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND clock_gettime@GLIBC_2.17 (16)
    ......
  4582: 0000000000794260    70 FUNC    WEAK   DEFAULT   12 _ZNSt15basic_streambufIwS

Version symbols section '.gnu.version' contains 4583 entries:
 Addr: 000000000045b508  Offset: 0x05b508  Link: 4 (.dynsym)
  000:   0 (*local*)       0 (*local*)       2 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)
  004:   3 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)
  008:   4 (GLIBC_2.3.2)   3 (GLIBC_2.2.5)   0 (*local*)       5 (GLIBC_2.14)
  ......
  040:   2 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)  10 (GLIBC_2.17)
  ......
  11e0:   1 (*global*)      1 (*global*)      1 (*global*)      1 (*global*)
  11e4:   1 (*global*)      1 (*global*)      1 (*global*)

Version needs section '.gnu.version_r' contains 6 entries:
 Addr: 0x000000000045d8d8  Offset: 0x05d8d8  Link: 5 (.dynstr)
  000000: Version: 1  File: ld-linux-x86-64.so.2  Cnt: 1
  0x0010:   Name: GLIBC_2.3  Flags: none  Version: 17
  0x0020: Version: 1  File: libgcc_s.so.1  Cnt: 3
  0x0030:   Name: GCC_3.0  Flags: none  Version: 13
  0x0040:   Name: GCC_3.3  Flags: none  Version: 11
  0x0050:   Name: GCC_4.2.0  Flags: none  Version: 10
  0x0060: Version: 1  File: libm.so.6  Cnt: 1
  0x0070:   Name: GLIBC_2.2.5  Flags: none  Version: 8
  0x0080: Version: 1  File: libpthread.so.0  Cnt: 2
  0x0090:   Name: GLIBC_2.3.2  Flags: none  Version: 15
  0x00a0:   Name: GLIBC_2.2.5  Flags: none  Version: 7
  0x00b0: Version: 1  File: libc.so.6  Cnt: 10
  0x00c0:   Name: GLIBC_2.8  Flags: none  Version: 19
  0x00d0:   Name: GLIBC_2.9  Flags: none  Version: 18
  0x00e0:   Name: GLIBC_2.17  Flags: none  Version: 16
  0x00f0:   Name: GLIBC_2.4  Flags: none  Version: 14
  0x0100:   Name: GLIBC_2.3.4  Flags: none  Version: 12
  0x0110:   Name: GLIBC_2.3  Flags: none  Version: 9
  0x0120:   Name: GLIBC_2.7  Flags: none  Version: 6
  0x0130:   Name: GLIBC_2.14  Flags: none  Version: 5
  0x0140:   Name: GLIBC_2.3.2  Flags: none  Version: 4
  0x0150:   Name: GLIBC_2.2.5  Flags: none  Version: 3
  0x0160: Version: 1  File: libdl.so.2  Cnt: 1
  0x0170:   Name: GLIBC_2.2.5  Flags: none  Version: 2
```

我们可以在 ELF 的 .dynsym 动态符号表中看到程序用于动态链接的所有导入导出符号，memcpy 和 clock_gettime 后面括号里的数字就是十进制的版本号（分别为 5 和 16），而我们需要格外关注的是下面的 .gnu.version 和 .gnu.version_r 符号版本信息段。

.gnu.version 表包含所有动态符号的版本信息，.dynsym 动态符号表中的每个符号都可以在 .gnu.version 中看到对应的条目（.dynsym 中一共 4583 个符号刚好与 .gnu.version 的结束位置 0x11e7 相等）。

从上面的输出可以看到 .gnu.version 表从 0x05b508 偏移量开始，我们可以看看对应偏移量的十六进制数据：

```

Offset(h) 00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F
0005B500                          00 00 00 00 02 00 03 00          ........
0005B510  03 00 03 00 03 00 03 00 04 00 03 00 00 00 05 00  ................
0005B520  03 00 03 00 06 00 00 00 03 00 07 00 08 00 08 00  ................
0005B530  03 00 09 00 03 00 03 00 0A 00 07 00 03 00 00 00  ................
0005B540  03 00 03 00 0B 00 07 00 03 00 03 00 00 00 07 00  ................
0005B550  00 00 03 00 03 00 03 00 03 00 0C 00 09 00 00 00  ................
0005B560  07 00 03 00 03 00 07 00 03 00 07 00 0C 00 00 00  ................
0005B570  0D 00 03 00 07 00 07 00 0E 00 0F 00 03 00 0D 00  ................
0005B580  03 00 03 00 03 00 03 00 02 00 03 00 03 00 10 00  ................
0005B590  03 00 00 00 03 00 07 00 08 00 07 00 07 00 03 00  ................
0005B5A0  03 00 0D 00 03 00 00 00 03 00 03 00 03 00 00 00  ................
```

.gnu.version 中的每个条目占用两个字节，其值为符号的版本，由此可以看到其中第 0x0b 个符号（也就是 .dynsym 表中的 memcpy@GLIBC_2.14 符号）的偏移量即为 0x05b51e（0x05b508 + 0x0b x 2），该偏移量的值 0x0005 也刚好和 .dynsym 表中的值对应，当然 clock_gettime 符号对应的偏移量 0x05b58e 的值 0x0010 同样也是如此。

下面关键的 .gnu.version_r 表示二进制程序实际依赖的库文件版本，从输出中也能看到 .gnu.version_r 表是按照不同的库文件进行分段显示的，每个条目占用 0x10 也就是 16 个字节，该表是从 0x05d8d8 偏移量开始，我们看看 GLIBC_2.17 也就是 0x05d9b8 处的十六进制数据：

```
Offset(h) 00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F
0005D9B0  B0 70 03 00 10 00 00 00 97 91 96 06 00 00 10 00  °p......—‘–.....
0005D9C0  BA 70 03 00 10 00 00 00 14 69 69 0D 00 00 0E 00  ºp.......ii.....
0005D9D0  C5 70 03 00 10 00 00 00 74 19 69 09 00 00 0C 00  Åp......t.i.....
0005D9E0  CF 70 03 00 10 00 00 00 13 69 69 0D 00 00 09 00  Ïp.......ii.....
0005D9F0  6A 70 03 00 10 00 00 00 17 69 69 0D 00 00 06 00  jp.......ii.....
0005DA00  DB 70 03 00 10 00 00 00 94 91 96 06 00 00 05 00  Ûp......”‘–.....
0005DA10  E5 70 03 00 10 00 00 00 72 19 69 09 00 00 04 00  åp......r.i.....
0005DA20  9A 70 03 00 10 00 00 00 75 1A 69 09 00 00 03 00  šp......u.i.....
0005DA30  8E 70 03 00 00 00 00 00 01 00 01 00 D8 03 00 00  Žp..........Ø...

```

.gnu.version_r 表中每个条目是 16 个字节的 Elfxx_Vernaux 结构体，其声明如下（Elfxx_Half 占用 2 个字节，Elfxx_Word 占用 4 个字节）：

```

typedef struct {
    Elfxx_Word    vna_hash;
    Elfxx_Half    vna_flags;
    Elfxx_Half    vna_other;
    Elfxx_Word    vna_name;
    Elfxx_Word    vna_next;
} Elfxx_Vernaux;
```

vna_hash 为 4 个字节的库名称（也就是上面的 GLIBC_2.17 字符串）的 hash 值，vna_other 为对应的 .gnu.version 表中符号的版本值，vna_name 指向库名称字符串的偏移量（也可以在 ELF 头中找到），vna_next 为下一个条目的位置（一般固定为 0x00000010）。

由上面的输出我们可以看到 GLIBC_2.17 对应的 0x05d9b8 处的开始的 4 个字节 vna_hash hash 值为 0x06969197，而 vna_other 的值 0x0010（输出里的 Version: 16）也与 .gnu.version 中 clock_gettime 符号的值一致。同样 GLIBC_2.14 也与 memcpy 符号的值相符。

## **修改 ELF 符号表**

由于 Linux 系统中的 LD 库（也就是 /lib64/ld-linux-x86-64.so.2 库）加载 ELF 时检查 .gnu.version_r 表中的符号，我们可以使用任何一款十六进制编辑器来修改 .gnu.version_r 表中的符号值来强制使用老版本的函数实现。

首先我们发现 .gnu.version_r 的 libc.so.6 段下面有 10 个条目，最后一个则是我们需要的 GLIBC_2.2.5 版本的符号（从上面的十六进制输出中我们可以看到该符号的偏移量为 0x05da28，vna_hash 值为 0x09691A75，vna_other 版本值为 0x0003，vna_name 字符串名称指向 0003708E 地址），因为这样我们才可以在不修改 ELF 文件大小的前提下直接将 libc.so.6 段下的其它高版本条目指向老版本条目的值。

例如 GLIBC_2.17 对应的 0x05d9b8 偏移量，我们可以直接将 vna_hash 值改为 GLIBC_2.2.5 的 0x09691A75 值，将 vna_other 改为 0003708E 值，为了保持和 .gnu.version 表中的版本值一致，这里我们就不修改 vna_other 值了。

对于 GLIBC_2.14 偏移量我们也修改成同样的值，修改保存之后的 ELF 文件再使用 readelf 命令检查就能看到变化了（只列出了修改的 .gnu.version-r 表）：

```

[root@centos6-dev ~]# readelf -sV tester
......
Version needs section '.gnu.version_r' contains 6 entries:
 Addr: 0x000000000045d8d8  Offset: 0x05e8d8  Link: 2 (.dynstr)
  000000: Version: 1  File: ld-linux-x86-64.so.2  Cnt: 1
  0x0010:   Name: GLIBC_2.3  Flags: none  Version: 17
  0x0020: Version: 1  File: libgcc_s.so.1  Cnt: 3
  0x0030:   Name: GCC_3.0  Flags: none  Version: 13
  0x0040:   Name: GCC_3.3  Flags: none  Version: 11
  0x0050:   Name: GCC_4.2.0  Flags: none  Version: 10
  0x0060: Version: 1  File: libm.so.6  Cnt: 1
  0x0070:   Name: GLIBC_2.2.5  Flags: none  Version: 8
  0x0080: Version: 1  File: libpthread.so.0  Cnt: 2
  0x0090:   Name: GLIBC_2.3.2  Flags: none  Version: 15
  0x00a0:   Name: GLIBC_2.2.5  Flags: none  Version: 7
  0x00b0: Version: 1  File: libc.so.6  Cnt: 10
  0x00c0:   Name: GLIBC_2.8  Flags: none  Version: 19
  0x00d0:   Name: GLIBC_2.9  Flags: none  Version: 18
  0x00e0:   Name: GLIBC_2.2.5  Flags: none  Version: 16
  0x00f0:   Name: GLIBC_2.4  Flags: none  Version: 14
  0x0100:   Name: GLIBC_2.3.4  Flags: none  Version: 12
  0x0110:   Name: GLIBC_2.3  Flags: none  Version: 9
  0x0120:   Name: GLIBC_2.7  Flags: none  Version: 6
  0x0130:   Name: GLIBC_2.2.5  Flags: none  Version: 5
  0x0140:   Name: GLIBC_2.3.2  Flags: none  Version: 4
  0x0150:   Name: GLIBC_2.2.5  Flags: none  Version: 3
  0x0160: Version: 1  File: libdl.so.2  Cnt: 1
  0x0170:   Name: GLIBC_2.2.5  Flags: none  Version: 2
```

## **patchelf 修改 ELF 文件**

一般的程序如果只使用了高版本 memcpy 的话，一般这样修改之后程序就可以运行了。但不巧我使用的第三方程序还使用了高版本 glibc 中的 clock_gettime，只是这样修改的话由于 CentOS 6 的 libc 2.12 库并没有提供 clock_gettime，运行时还是会报错。

这个时候我们就需要请出大杀器 PatchELF 了，这个小工具由 NixOS 团队开发，可以直接增加、删除、替换 ELF 文件依赖的库文件，使用起来也非常简单。

检出 PatchELF 的源代码，按照 GitHub 仓库上介绍的步骤编译安装就可以使用了（一般发行版自带的 patchelf 工具版本较老不支持一些新的功能）。

虽然 CentOS 6 的 libc 库没有提供 clock_gettime 实现，但好在 glibc 自带的 librt 库里还是提供了的，因此我们可以使用 patchelf 工具修改原版的程序文件，让程序优先加载 librt 库，这样程序就能正确加载 clock_gettime 符号了：

```
[root@centos6-dev ~]# patchelf --add-needed librt.so.1 tester
```

然后按照上面介绍的方法用十六进制编辑器修改新生成的 ELF 文件的 .gnu.version_r 表（因为 patchelf 运行之后新 ELF 文件的符号表就和之前的不一样了），将 GLIBC_2.17 和 GLIBC_2.14 统一改为 GLIBC_2.2.5 符号，保存 ELF 文件之后就可以看到效果了：

```

[root@centos6-dev ~]# ldd tester
        linux-vdso.so.1 =>  (0x00007fffc17ee000)
        librt.so.1 => /lib64/librt.so.1 (0x00007f7f84dca000)
        libpthread.so.0 => /lib64/libpthread.so.0 (0x00007f7f84bad000)
        libOpenCL.so.1 => /usr/lib64/libOpenCL.so.1 (0x00007f7f8498f000)
        libdl.so.2 => /lib64/libdl.so.2 (0x00007f7f8478b000)
        libm.so.6 => /lib64/libm.so.6 (0x00007f7f84507000)
        libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x00007f7f842f1000)
        libc.so.6 => /lib64/libc.so.6 (0x00007f7f83f5d000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f7f84fd2000)
```

从 ldd 命令的输出中可以看到修改后的程序会加载 librt 库，而且也没有 glibc 版本的报错了，经过测试程序运行起来也没有问题了。

