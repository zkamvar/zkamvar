---
title: I C Bugs
author: Zhian N. Kamvar
date: '2018-01-14'
slug: i-c-bugs
categories:
  - R
tags:
  - CRAN
  - poppr
  - R
  - C
  - bug
  - lldb
  - clang
banner: img/banners/Halyomorpha_halys_lab.jpg
---

<figure>
<img width="50%" src="img/banners/Halyomorpha_halys_lab.jpg" alt="A Brown marmorated stink bug female from a laboratory colony on a common bean leaf, photographed in the laboratory of Fondazione Edmund Mach, Italy." longdesc = "https://commons.wikimedia.org/wiki/File:Halyomorpha_halys_lab.jpg" />
<figcaption width="75%">
A Brown marmorated stink bug female from a laboratory colony on a common bean leaf, photographed in the laboratory of Fondazione Edmund Mach, Italy.<br>
URL: https://commons.wikimedia.org/wiki/File:Halyomorpha_halys_lab.jpg
</figcaption>
</figure>

Occasionally, I hear people complain about the strict policies of [CRAN], but I
for one quite apprecieate them, especially when dealing with hidden errors in
compiled code. 

Not twenty-four hours after I had submitted [poppr version 2.6.0](https://zkamvar.github.io/blog/poppr-2-6-0-better-network-plotting/) 
to [CRAN] did I receive an email from none other than Brian D. Ripley. If you
are unfamiliar with the name, then know that it tends to [strike fear] in the
hearts of even the [most steeled] [R veterans]:


```r
library("fortunes")
fortune("Ripleyed")
```

```
## 
## And the fear of getting Ripleyed on the mailing list also makes me think, read,
## and improve before submitting half baked questions to the list.
##    -- Eric Kort
##       R-help (January 2006)
```

This was not the first time I got an email from Ripley (and probably won't be 
the last). More often than not, the emails I get from him alert me to little 
things that have gone wrong in memory allocation for code that I've written in C.
The email I got from him on January 9th was informing me that there was a new
note from the [clang address sanitizer] saying that there was a [potential
memory leak in `bitwise_distance.c`](https://github.com/grunwaldlab/poppr/issues/169):

> Please correct ASAP and before Jan 24 to safely retain the package on CRAN.


# Fixing the error

I had initially attempted to try to reproduce the error using a [docker container
built with the address sanitizer](https://hub.docker.com/r/zkamvar/poppr-devel-docker/~/dockerfile/),
but quickly became frustrated with it due to errors unrelated to poppr. My next
plan of action was to inspect the code using a debugger. I found [Kevin Ushey's
blog post on debugging with LLDB](http://kevinushey.github.io/blog/2015/04/13/debugging-with-lldb/) 
and began to go to work. 

The error occurred [at the end of a "while" loop](https://github.com/grunwaldlab/poppr/blob/453f08f56574b778775988e7fce2650d4a37ef5f/src/bitwise_distance.c#L674) that was written four years ago. Here's what the code looks like without
the comments:

```c
while(next_missing_index_i < nap1_length && next_missing_i < (k+1)*8 && next_missing_i >= (k*8) )
{
  mask = 1 << (next_missing_i%8); 
  if(missing_match)
  {
    tmp_sim_set |= mask; // Force the missing bit to match
  }
  else
  {
    tmp_sim_set &= ~mask; // Force the missing bit to not match
  }
  // Find the index of the next missing value in sample i.
  next_missing_index_i++; 
  next_missing_i = (int)INTEGER(R_nap1)[next_missing_index_i] - 1; // <--- BUG!
}       
```

I won't go into detail what this code does, but the reason why it triggered a 
warning from the address sanitizer is because of a [fencepost error]. The loop 
monitors `next_missing_index_i` and repeats the operation until it is equal to
`nap1_length`, which happens to be the length of the array it indexes. Because
the index is incremented before the array is indexed, the last step of the loop
always attempts to access a bit of memory that wasn't allocated. 

> Note: to debug, be sure to change your optimization flags to `-O0` for your compiler.
> You can do this in your `~/.R/Makevars` file:
> ```sh
> CFLAGS +=         -O0 -Wall -pipe -pedantic -std=gnu99 -fopenmp
> CXXFLAGS +=       -O0 -Wall -pipe -Wno-unused -pedantic
> CXX1XFLAGS +=     -O0 -pipe
> PKG_CXX1XFLAGS += -O0 -pipe
> ```

Using the debugger was surprisingly easy. My process was to tweak the C code and
then start R with the debugger, and install poppr:

```lldb
17:47:09~/Documents/poppr (master)> R -d lldb
*** Further command line arguments ('--no-save --no-restore-data ') disregarded
*** (maybe use 'run --no-save --no-restore-data ' from *inside* lldb)

(lldb) target create "/Library/Frameworks/R.framework/Resources/bin/exec/R"
Current executable set to '/Library/Frameworks/R.framework/Resources/bin/exec/R' (x86_64).
(lldb) run --no-save --no-restore-data -q
Process 10723 launched: '/Library/Frameworks/R.framework/Resources/bin/exec/R' (x86_64)
> devtools::install()
...
> library("poppr")
...
>
```

After that, I could load my test data and, before I ran `bitwise.dist()`, I hit
<kbd>ctrl+c</kbd>, which took me back into lldb:

```c
> Process 10723 stopped
* thread #1: tid = 0x8251a, 0x00007fff60f01fca libsystem_kernel.dylib`__select + 10, stop reason = signal SIGSTOP
    frame #0: 0x00007fff60f01fca libsystem_kernel.dylib`__select + 10
libsystem_kernel.dylib`__select:
->  0x7fff60f01fca <+10>: jae    0x7fff60f01fd4            ; <+20>
    0x7fff60f01fcc <+12>: movq   %rax, %rdi
    0x7fff60f01fcf <+15>: jmp    0x7fff60ef90dd            ; cerror
    0x7fff60f01fd4 <+20>: retq
(lldb) 
```

From there, to get it to breakpoint on the specific line, I typed:

```c
(lldb) b bitwise_distance.c:674
Breakpoint 1: where = poppr.so`.omp_outlined..4 + 1471 at bitwise_distance.c:674, address = 0x000000010f72435f
(lldb) c
Process 10723 resuming

> 
```

From there, when I ran `bitwise.dist()` it always stopped whenever it ran into
that point:

```c
Process 11033 stopped
* thread #4: tid = 0x84b9a, 0x0000000100f11394 poppr.so`.omp_outlined..4(.global_tid.=0x00007000058fcd70, .bound_tid.=0x00007000058fcd68, i=0x00007ffeefbfcafc, R_gen=0x00007ffeefbfcb70, R_chr_symbol=0x00007ffeefbfcb80, R_nap_symbol=0x00007ffeefbfcb78, nap1_length=0x00007ffeefbfcb34, R_nap1=0x00007ffeefbfcb40, chr_length=0x00007ffeefbfcb2c, R_chr1_1=0x00007ffeefbfcb60, R_chr1_2=0x00007ffeefbfcb58, missing_match=0x00007ffeefbfcb18, only_differences=0x00007ffeefbfcb14, distance_matrix=0x00007ffeefbfcae8) + 1428 at bitwise_distance.c:674, stop reason = breakpoint 1.1
    frame #0: 0x0000000100f11394 poppr.so`.omp_outlined..4(.global_tid.=0x00007000058fcd70, .bound_tid.=0x00007000058fcd68, i=0x00007ffeefbfcafc, R_gen=0x00007ffeefbfcb70, R_chr_symbol=0x00007ffeefbfcb80, R_nap_symbol=0x00007ffeefbfcb78, nap1_length=0x00007ffeefbfcb34, R_nap1=0x00007ffeefbfcb40, chr_length=0x00007ffeefbfcb2c, R_chr1_1=0x00007ffeefbfcb60, R_chr1_2=0x00007ffeefbfcb58, missing_match=0x00007ffeefbfcb18, only_differences=0x00007ffeefbfcb14, distance_matrix=0x00007ffeefbfcae8) + 1428 at bitwise_distance.c:674
   671 	          }
   672 	          // Find the index of the next missing value in sample i.
   673 	          next_missing_index_i++;
-> 674 	          next_missing_i = (int)INTEGER(R_nap1)[next_missing_index_i] - 1;
   675 	        }
   676 	        // Repeat for j
   677 	        while(next_missing_index_j < nap2_length && next_missing_j < (k+1)*8 && next_missing_j >= (k*8))
(lldb)
```

I could get a glimpse of all the variables if I typed `frame variable` (you can also type "fr v"):

```c
(lldb) frame variable
(int *const __restrict) .global_tid. = 0x00007000058fcd70
(int *const __restrict) .bound_tid. = 0x00007000058fcd68
(int &) i = 0x00007ffeefbfcafc (&i = 1)
(SEXP &) R_gen = 0x00007ffeefbfcb70 (&R_gen = 0x000000010fc616a0)
(SEXP &) R_chr_symbol = 0x00007ffeefbfcb80 (&R_chr_symbol = 0x000000010bab7578)
(SEXP &) R_nap_symbol = 0x00007ffeefbfcb78 (&R_nap_symbol = 0x00000001024126d0)
(int &) nap1_length = 0x00007ffeefbfcb34 (&nap1_length = 2)
(SEXP &) R_nap1 = 0x00007ffeefbfcb40 (&R_nap1 = 0x000000010dc52e78)
(int &) chr_length = 0x00007ffeefbfcb2c (&chr_length = 4)
(SEXP &) R_chr1_1 = 0x00007ffeefbfcb60 (&R_chr1_1 = 0x000000010dc53238)
(SEXP &) R_chr1_2 = 0x00007ffeefbfcb58 (&R_chr1_2 = 0x000000010dc51cf8)
(int &) missing_match = 0x00007ffeefbfcb18 (&missing_match = 1)
(int &) only_differences = 0x00007ffeefbfcb14 (&only_differences = 0)
(int **&) distance_matrix = 0x00007ffeefbfcae8: {
  &distance_matrix = 0x0000000100a6fa50
}
(int) .omp.iv = 0
(int) .capture_expr. = 1
(int) .capture_expr. = 0
(int) j = 0
(int) .omp.lb = 0
(int) .omp.ub = 0
(int) .omp.stride = 1
(int) .omp.is_last = 1
(int) j = 0
(int) cur_distance = 0
(SEXP) R_chr2_1 = 0x000000010fa681d8
(SEXP) R_chr2_2 = 0x000000010dfe1898
(SEXP) R_nap2 = 0x000000010c7db4e0
(int) next_missing_index_j = -1
(int) next_missing_j = -1
(int) next_missing_index_i = 1
(int) next_missing_i = 4
(zygosity) set_1 = (c1 = '\0', c2 = '\x87', ch = '\x87', cd = '\0', cr = 'x')
(zygosity) set_2 = (c1 = '@', c2 = '�', ch = '�', cd = '@', cr = '\x03')
(char) tmp_sim_set = '\x94'
(int) k = 0
(char) mask = '\x10'
(int) nap2_length = 0
```

> Notice how we have a lot of weird characters up there? That's because this 
> function deals with data stored in bits. [You can actually tell lldb to display
> characters as bits](https://lldb.llvm.org/varformats.html):
>
> ```c
> type format add --format binary char
> ```
> 
> Now the values can actually be interpreted!
>
> ```c
> (zygosity) set_1 = (c1 = 0b00000000, c2 = 0b10000111, ch = 0b10000111, cd = 0b00000000, cr = 0b01111000)
> (zygosity) set_2 = (c1 = 0b01000000, c2 = 0b11111100, ch = 0b10111100, cd = 0b01000000, cr = 0b00000011)
> (char) tmp_sim_set = 0b10010100
> (int) k = 0
> (char) mask = 0b00010000
> ```


We can see that `next_missing_i` is set to 4, `next_missing_index_i` is currently
set to 1 and the `nap1_length` is set to 2. If we type "n", it will execute the
current line. We can then see the new value of `next_missing_i`

```c
(lldb) print next_missing_i
(int) $1 = 17
```

Instead of going line by line, you can type "c" to go to the next breakpoint and
execute the code there. This happens to be the point of the error. You can see
that if we execute line 674 and then print `next_missing_i`:

```c
(lldb) print next_missing_i
(int) $2 = 536870927
```

The data I fed into this function only has 24 possible positions, so this huge
number is the memory leak.

# Solution

The solution I eventually came up with was to wrap it in another check:

```c
  // Find the index of the next missing value in sample i.
  next_missing_index_i++; 
  if (next_missing_index_i < nap1_length){
    next_missing_i = (int)INTEGER(R_nap1)[next_missing_index_i] - 1;
  }
```

Is it elegant? No. Did it get the job done? Yes. I had originally moved the 
assignment step to the top of the loop, but had tests fail that way and I 
couldn't figure out what was going on with them. 

# Lesson learned

This error has been in the code for four years. It wasn't particularly malicious
since it wasn't overwriting this space of memory, but it's the kind of error
that could only be detected by these address sanitizers routinely used by the
CRAN team. In fact, the Valgrind sanitizer had pointed out this very error, but,
I had (wrongly) thought that this was due to the fact that [it was in the middle
of a parallel processing statement since the memory was only "possibly" lost](https://github.com/grunwaldlab/poppr/blob/379c42ba52afb9e2ce2bbf9141bf540cb16d0a37/cran-comments.md#valgrind).
Sometimes these errors are obvious and other times it takes you over a year and
a half to find these bugs and squish them).



[CRAN]: https://cran.r-project.org/package=poppr
[strike fear]: http://www.gastonsanchez.com/visually-enforced/opinion/2013/12/10/I-violated-CRAN-Repository-Policies/
[most steeled]: https://twitter.com/xieyihui/status/290525069300617218
[R veterans]: https://twitter.com/hadleywickham/status/276022791839571968
[clang address sanitizer]: https://www.stats.ox.ac.uk/pub/bdr/memtests/README.txt
[fencepost error]: https://en.wikipedia.org/wiki/Off-by-one_error#Fencepost_error
