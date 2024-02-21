# memtester
                                       
## About memtester
  
memtester is a utility for testing the memory subsystem in a computer to
determine if it is faulty. The original source was by Simon Kirby
<sim@stormix.com>. Charles Cazabon has by this time completely rewritten the
original source, and added many additional tests to help catch
borderline memory. Charles Cazabon also rewrote the original tests (which catch
mainly memory bits which are stuck permanently high or low) so that
they run approximately an order of magnitude faster.

The version 4 rewrite was mainly to accomplish three things:

1. the previous code was basically a hack, and was ugly.
2. to make the code more portable.  The previous version required some
    hackery to compile on some systems.
3. to make the code fully 64-bit aware.  The previous version worked
    on 64-bit systems, but did not fully stress the memory subsystems
    on them -- this version should be better at stress-testing 64-bit
    systems.
         
## Building memtester
  
memtester uses CMake as build tool. The basic build can be done as follows:

```bash
mkdir build
cd build
cmake ..
make
#optional
sudo make install 
```

Memtester 4 has been sucessfully build on the following systems:

  - HP Tru64 Unix 4.0g (Alpha)
  - HP Tru64 Unix 5.1b (Alpha)
  - HP-UX 11i 11.11 (PA-RISC)
  - HP-UX 11i 11.23 (64-bit Itanium)
  - Debian GNU/Linux 3.0 (various)
  - other 32-bit Linux (RedHat, SuSE, Ubuntu, etc) (various)
  - RedHat Enterprise Linux/CentOS (64-bit AMD Opteron)
  - FreeBSD 4.9 (32-bit Intel)
  - FreeBSD 5.1 (64-bit Alpha)
  - NetBSD 1.6 (32-bit Intel)
  - Darwin (OS X) 7.5.0 (32-bit PowerPC)
  - OS X Leopard/Panther/whatever -- 32- or 64-bit, PPC or x86

It should, however, work on other Unix-like systems.
If you have trouble building memtester on your system, please report it
to me so it can be fixed.

## Using memtester
  
Usage is simple for the basic case.  As root, run the resulting memtester
binary with the following commandline:
```bash
memtester <memory> [runs]
``` 
where <memory> is the amount of memory to test, in megabytes by default.
You can optionally include a suffix of B, K, M, or G (for bytes, 
kilobytes, megabytes, and gigabytes respectively).
[runs] is an optional limit to the number of runs through all tests.
    
An optional "-p physaddr" argument available to cause memtester to test
memory starting at a specific physical memory address (by mmap(2)ing 
a device file representing physical memory (/dev/mem by default, but can
be specified with the "-d device" option) starting at an offset of 
`physaddr`, which is given in hex).

Note:  the memory specified will be overwritten during testing; you 
therefore *cannot* specify a region belonging to the kernel or other
applications without causing the other process or entire system to
crash).  If you use this option, it is up to you to ensure the specified
memory is safe to overwrite.  That makes this option mostly of use for
testing memory-mapped I/O devices and similar.  Thanks to Allon Stern
for the idea behind this feature.  For example, if you want to test a
bank of RAM or device which is 64kbytes in size and starts at physical
address 0x0C0000 through the normal /dev/mem, you would run memtester as 
follows:
```bash
memtester -p 0x0c0000 64k [runs]
``` 
If instead that device presented its memory as /dev/foodev at offset
0, you would run memtester instead as follows:
```bash
memtester -p 0 -d /dev/foodev 64k [runs]
``` 

Note that the "-d" option can only be specified in combination with "-p".
    
memtester must run as user root so that it can lock its pages into 
memory. If memtester fails to lock its pages, it will issue a warning and 
continue regardless.  Testing without the memory being locked is generally
very slow and not particularly accurate, as you'll end up testing the same
memory over and over as the system swaps the larger region.
   
## About the Tests

The following tests are from the original version, updated simply for speed
and rewritten to fit the new framework of the program.  These tests will
mainly catch memory errors due to bad bits which are permanently stuck high
or low:
- Random value
- XOR comparison
- SUB comparison
- MUL comparison
- DIV comparison
- OR comparison
- AND comparison
  
The following tests were implemented by Charles Cazabon, and will do a slightly better job
of catching flaky bits, which may or may not hold a true value:
- Sequential Increment
- Block Sequential
- Solid Bits

The remaining tests were also implemented by Charles Cazabon, and are designed to catch
bad bits which are dependent on the current values of surrounding bits in either
the same word32, or in the preceding and succeeding word32s.
- Bit Flip
- Checkerboard
- Walking Ones
- Walking Zeroes
- Bit Spread

There is also a test (Stuck Address) which is run first.  It determines if the 
memory locations the program attempts to access are addressed properly or not.  
If this test reports errors, there is almost certainly a problem somewhere in 
the memory subsystem.  Results from the rest of the tests cannot be considered 
accurate if this test fails:
- Stuck Address

## Bugs

Please report bugs by opening an issue or submitting a PR on the Github project.

## Platform Compatibility Notes

As mentioned, memtester v4 was tested with a wide variety of Unix- and
Unix-like systems.  However, at least two issues with ancient HP/UX
versions, and one with Tru64, have come up since.

HP/UX versions prior to v11 do not support mlock() and will fail with an
invalid syscall error at runtime.  If you're building on HP/UX v10.20 or
similar, change `int do_mlock = 1` around line 128 of memtester.c to
`int do_mlock = 0` and rebuild.
They also are missing strtoull() from C99.  To fix this build error,
add `#define strtoull __strtoull` to the top of memtester.c; this should
at least work with a GCC toolchain.

Tru64 is missing strtoull() from C99.  To fix this build error when using
cc, add `#define strtoull strtoul` to the top of memtester.c; this is
safe because long on Tru64 is 64-bits.  This build problem could be dealt
with automatically, but I don't particularly want to invest the necessary
time, because Tru64 went out of support in 2012.  In addition, I no longer
have access to a build machine for testing on this platform.


## Copyright
  
by Charles Cazabon <charlesc-memtester@pyropus.ca>

forked in 2024 by Benedikt Bergenthal <mail@benedikt-bergenthal.de>

Copyright 1999 Simon Kirby.

Version 2 Copyright 1999 Charles Cazabon.

Version 3 not publicly released.

Version 4 rewrite:

Copyright 2004-2020 Charles Cazabon.

2024 Benedikt Bergenthal <mail@benedikt-bergenthal.de>

Licensed under the terms of the GNU General Public License version 2 (only).

See the file COPYING for details.