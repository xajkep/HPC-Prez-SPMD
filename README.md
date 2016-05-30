Cyrill Zundler &  Benoît Zuckschwerdt

# HPC - Intel SPMD

## Installation

Installation réalisé sur Debian 8.4 (64bits).
~~~sh
$ uname -a
Linux eternia 3.16.0-4-amd64 #1 SMP Debian 3.16.7-ckt25-2 (2016-04-08) x86_64 GNU/Linux
~~~

### Instructions

Binaries are available for Windows, Mac OS X and Linux at the address: [http://ispc.github.io/downloads.html]()

~~~sh
wget http://sourceforge.net/projects/ispcmirror/files/v1.9.0/ispc-v1.9.0-linux.tar.gz/download
mv download ispc-v1.9.0-linux.tar.gz
tar xvzf ispc-v1.9.0-linux.tar.gz
~~~

**Note:** Sources available on the following github repository: [https://github.com/ispc/ispc/]()

## Exemple

### Bucket Sort

#### Compilation

~~~sh
$ cat Makefile

EXAMPLE=sort
CPP_SRC=sort.cpp sort_serial.cpp
ISPC_SRC=sort.ispc
ISPC_IA_TARGETS=sse2-i32x4,sse4-i32x8,avx1-i32x8,avx2-i32x8
ISPC_ARM_TARGETS=neon

include ../common.mk

$ make
/bin/mkdir -p objs/
ispc -O2 --arch=x86-64 --target=sse2-i32x4,sse4-i32x8,avx1-i32x8,avx2-i32x8 sort.ispc -o objs/sort_ispc.o -h objs/sort_ispc.h
clang++ sort.cpp -Iobjs/ -O2 -m64 -c -o objs/sort.o
clang++ sort_serial.cpp -Iobjs/ -O2 -m64 -c -o objs/sort_serial.o
clang++ ../tasksys.cpp -Iobjs/ -O2 -m64 -c -o objs/tasksys.o
clang++ -Iobjs/ -O2 -m64 -o sort objs/sort.o objs/sort_serial.o objs/tasksys.o objs/sort_ispc.o objs/sort_ispc_sse2.o objs/sort_ispc_sse4.o objs/sort_ispc_avx.o objs/sort_ispc_avx2.o -lm -lpthread -lstdc++
~~~

#### Execution

~~~sh
$ ./sort
[====================== 100 % =====================]
[sort ispc]:	[4999.266] million cycles
[====================== 100 % =====================]
[sort ispc + tasks]:	[3171.516] million cycles
[====================== 100 % =====================]
[sort serial]:		[12391.010] million cycles
				(2.48x speedup from ISPC, 3.91x speedup from ISPC + tasks)
~~~

We can see a performance gain of 248% for ISPC and of 391% for ISPC with tasking model compared with CPP code.


## Liens

* [http://ispc.github.io/]()
* [https://github.com/ispc/ispc/]()
* [https://software.intel.com/sites/default/files/ISPC-a-SPMD-Compiler-for-Xeon-Phi-CPU.pdf]()
* [https://en.wikipedia.org/wiki/Bucket_sort]()
