#Challenges
https://wiki.winehq.org/Wine_Developer%27s_Guide/Kernel_modules
##The problem 1 - Memory Management
This is an extras from documentation 

~2 Detailed memory management
As already explained in a previous chapter (see Memory management for details), Wine creates every 32-bit Windows process in its own 32-bit address space. Wine also tries to map at the relevant addresses what Windows would do. There are however a few nasty bits to look at.
2.1 Implementation
Wine (with a bit of black magic) is able to map the main module at it's desired address (likely 0x400000), to create the process heap, its stack (as a Windows executable can ask for a specific stack size), Wine simply use the initial stack of the ELF executable for its initialisation, but creates a new stack (as a Win32 one) for the main thread of the executable. Wine also tries to map all native DLLs at their desired address, so that no relocation has to be performed.

Wine also implements the shared heap so native win9x DLLs can be used. This heap is always created at the SYSTEM_HEAP_BASE address or 0x80000000 and defaults to 16 megabytes in size.

There are a few other magic locations. The bottom 64k of memory is deliberately left unmapped to catch null pointer dereferences. The region from 64k to 1mb+64k are reserved for DOS compatibility and contain various DOS data structures. Finally, the address space also contains mappings for the Wine binary itself, any native libraries Wine is using, the glibc malloc arena and so on.

2.2 Laying out the address space
Up until about the start of 2004, the Linux address space very much resembled the Windows 9x layout: the kernel sat in the top gigabyte, the bottom pages were unmapped to catch null pointer dereferences, and the rest was free. The kernels mmap algorithm was predictable: it would start by mapping files at low addresses and work up from there.

The development of a series of new low level patches violated many of these assumptions, and resulted in Wine needing to force the Win32 address space layout upon the system. This section looks at why and how this is done.

The exec-shield patch increases security by randomizing the kernels mmap algorithms. Rather than consistently choosing the same addresses given the same sequence of requests, the kernel will now choose randomized addresses. Because the Linux dynamic linker (ld-linux.so.2) loads DSOs into memory by using mmap, this means that DSOs are no longer loaded at predictable addresses, so making it harder to attack software by using buffer overflows. It also attempts to relocate certain binaries into a special low area of memory known as the ASCII armor so making it harder to jump into them when using string based attacks.

Prelink is a technology that enhances startup times by precalculating ELF global offset tables then saving the results inside the native binaries themselves. By grid fitting each DSO into the address space, the dynamic linker does not have to perform as many relocations so allowing applications that heavily rely on dynamic linkage to be loaded into memory much quicker. Complex C++ applications such as Mozilla, OpenOffice and KDE can especially benefit from this technique.

The 4G VM split patch was developed by Ingo Molnar. It gives the Linux kernel its own address space, thereby allowing processes to access the maximum addressable amount of memory on a 32-bit machine: 4 gigabytes. It allows people with lots of RAM to fully utilise that in any given process at the cost of performance: the reason behind giving the kernel a part of each processes address space was to avoid the overhead of switching on each syscall.

Each of these changes alter the address space in a way incompatible with Windows. Prelink and exec-shield mean that the libraries Wine uses can be placed at any point in the address space: typically this meant that a library was sitting in the region that the EXE you wanted to run had to be loaded (remember that unlike DLLs, EXE files cannot be moved around in memory). The 4G VM split means that programs could receive pointers to the top gigabyte of address space which some are not prepared for (they may store extra information in the high bits of a pointer, for instance). In particular, in combination with exec-shield this one is especially deadly as it's possible the process heap could be allocated beyond ADDRESS_SPACE_LIMIT which causes Wine initialization to fail.

The solution to these problems is for Wine to reserve particular parts of the address space so that areas that we don't want the system to use will be avoided. We later on (re/de)allocate those areas as needed. One problem is that some of these mappings are put in place automatically by the dynamic linker: for instance any libraries that Wine is linked to (like libc, libwine, libpthread etc) will be mapped into memory before Wine even gets control. In order to solve that, Wine overrides the default ELF initialization sequence at a low level and reserves the needed areas by using direct syscalls into the kernel (i.e. without linking against any other code to do it) before restarting the standard initialization and letting the dynamic linker continue. This is referred to as the preloader and is found in loader/preloader.c.

Once the usual ELF boot sequence has been completed, some native libraries may well have been mapped above the 3gig limit: however, this doesn't matter as 3G is a Windows limit, not a Linux limit. We still have to prevent the system from allocating anything else above there (like the heap or other DLLs) though so Wine performs a binary search over the upper gig of address space in order to iteratively fill in the holes with MAP_NORESERVE mappings so the address space is allocated but the memory to actually back it is not. This code can be found in libs/wine/mmap.c:reserve_area
~
