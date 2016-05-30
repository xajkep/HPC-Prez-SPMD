Cyrill Zundler &  Benoît Zuckschwerdt

# HPC - Intel SPMD

## Installation

Installation and tests performed on Debian 8.4 (64-bit).
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

## Examples

### Bucket Sort

The bucket sort is a sorting algorithm  that works by distributing the elements of an array into a number of buckets. Each buckets is then sorted individually.

Pseudocode:
~~~pseudo
function bucketSort(array, n) is
  buckets ← new array of n empty lists
  for i = 0 to (length(array)-1) do
    insert array[i] into buckets[msbits(array[i], k)]
  for i = 0 to n - 1 do
    nextSort(buckets[i]);
  return the concatenation of buckets[0], ...., buckets[n-1]
~~~

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

We can see a performance gain of 248% for ISPC and of 391% for ISPC with tasking model compared to the serial run.

### Mandelbrot

The Mandelbrot set is the set of complex numbers c for with the function  ![](images/mandelbrot_function.png
)
does not diverges when iterated from *z = 0*.

This implementation parallelizes across
cores using tasks. On Linux a pthreads-based task system is used (tasks_pthreads.cpp).

#### Compilation

~~~sh
$ cat Makefile

EXAMPLE=mandelbrot
CPP_SRC=mandelbrot.cpp mandelbrot_serial.cpp
ISPC_SRC=mandelbrot.ispc
ISPC_IA_TARGETS=sse2-i32x4,sse4-i32x8,avx1-i32x16,avx2-i32x16
ISPC_ARM_TARGETS=neon

include ../common.mk

$ make
/bin/mkdir -p objs/
ispc -O2 --arch=x86-64 --target=sse2-i32x4,sse4-i32x8,avx1-i32x16,avx2-i32x16 mandelbrot.ispc -o objs/mandelbrot_ispc.o -h objs/mandelbrot_ispc.h
clang++ mandelbrot.cpp -Iobjs/ -O2 -m64 -c -o objs/mandelbrot.o
clang++ mandelbrot_serial.cpp -Iobjs/ -O2 -m64 -c -o objs/mandelbrot_serial.o
clang++ ../tasksys.cpp -Iobjs/ -O2 -m64 -c -o objs/tasksys.o
clang++ -Iobjs/ -O2 -m64 -o mandelbrot objs/mandelbrot.o objs/mandelbrot_serial.o objs/tasksys.o objs/mandelbrot_ispc.o objs/mandelbrot_ispc_sse2.o objs/mandelbrot_ispc_sse4.o objs/mandelbrot_ispc_avx.o objs/mandelbrot_ispc_avx2.o -lm -lpthread -lstdc++
~~~

#### Execution

~~~sh
$ ./mandelbrot
@time of ISPC run:			[27.989] million cycles
@time of ISPC run:			[26.998] million cycles
@time of ISPC run:			[26.813] million cycles
[mandelbrot ispc]:		[26.813] million cycles
Wrote image file mandelbrot-ispc.ppm
@time of serial run:			[236.890] million cycles
@time of serial run:			[236.715] million cycles
@time of serial run:			[236.750] million cycles
[mandelbrot serial]:		[236.715] million cycles
Wrote image file mandelbrot-serial.ppm
				(8.83x speedup from ISPC)
~~~

On 10 runs we have obtained a average of performance gain of *862.1%* compared to the serial run.

#### Result

![](images/mandelbrot-ispc.png)


## Liens

* [http://ispc.github.io/]()
* [https://github.com/ispc/ispc/]()
* [https://software.intel.com/sites/default/files/ISPC-a-SPMD-Compiler-for-Xeon-Phi-CPU.pdf]()
* [https://en.wikipedia.org/wiki/Bucket_sort]()
* [https://fr.wikipedia.org/wiki/Ensemble_de_Mandelbrot]()
