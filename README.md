# Virtual Addressing Simulator (vasim)

## Objectives

Understand how virtual memory page tables and paging work.  
* How a referenced virtual address is broken into fields based on page size.
* How the fields are used to obtain the physical address for the reference.
* How to tell if the reference is to a missing page, which raises a page fault.
* How to select a victim page to evict to free its page frame: use the FIFO algorithm but understand its limitations.
* How to reallocate the victim's frame to hold the missing page: page replacement.
* How to update page table structures for a page fault and eviction (if needed). 
* Determine how many page faults result from an address reference trace.

Understand the implications of virtual paging for performance:
* Page faults and page replacement are expensive.
* Page fault must execute handler software and (in a real system) likely read/write from disk.
* Better victim selection (e.g., LRU replacement) can reduce the page fault rate.

## Overview
 
Your task is to complete the virtual memory simulation in vasim.c.

The parameters are fixed.   The system has a 24-bit virtual address space with 4KiB pages (KiB is kilobyte: 1024 bytes).
The physical memory holds a total
of 64 pages.  (Real machines have many more.)    The page table is a flat array that maps
virtual page numbers (VPN) to page frame numbers (PFNs), also called physical page numbers (PPNs).

Allocation and victim selection
(replacement policy) in this simulation are trivial: simple FIFO.   It works like this.   Allocate empty
frames in order of PFN (PPN).   Then, when they are all allocated, go back to PFN 0 and start choosing
victims in order of PFN.   Repeat as needed.  In this way, each victim
occupies the frame that was allocated first, i.e., the oldest among the current mappings.
That is First In First Out (FIFO).

## vasim

The `vasim` program has three command line arguments:
-  -f _filename_, 
- -n _access_limit_, and
- -v for verbose output.

The -f argument is required.   it specifies the name of a trace file.   The other arguments are optional. The -n flag stops simulation after _access_limit_ accesses are processed
from the tracefile.   The -v argument generates more verbose output.   

The `main()` is provided for you.   It processes the arguments and opens the trace file with name *filename*.     If `-v` is specified, it sets a global variable (flag) called *verbose* to 1.

The main loop processes each address of the trace in sequence, up to a limit
of *access_limit* entries.  
If the *verbose* flag is set it prints some info
about each reference.

## What to do

**vasim.c**
 
See the TODO comments in `vasim.c` to see where to put your code.  Fill in the two helper functions:

* **get_vpn**: returns the virtual page number from a virtual address

* **allocate_phys_page**:  page fault handler.    It is called only when a requested virtual
page (VPN) is not present, i.e., it is missing, i.e., the VPN does not have a valid mapping in the page table.  The two arguments
are a pointer to the page table and the VPN of the missing page.  It allocates a frame
(a physical page) to hold the missing page; if all frames are occupied, it selects a victim page to replace and reallocates the victim's
frame.   Then it updates the 
page table to create a valid mapping of the VPN to the allocated frame, invalidating the old mapping (if any).    Use the FIFO algorithm summarized above and in class.

In `allocate_phys_page`, if you decide to replace a victim and the *verbose* flag is set, then call the function `print_replacement()`
to print the event to stdout.

---

**Try it out**

Run make to build `vasim.c` in the usual way.   It won't work until your code is in place.

We provide two trace files: row_trace.txt and col_trace.txt.

Run the simulator on the command line:
```bash
./vasim -f [tracefile] -n 1024
```

**Example output:**
```
./vasim -f row_trace.txt -n 1024
Running for 1024 accesses

Accesses 1024: Hits: 1015 Faults: 9
```

---

**Local Testing and Submission**

```bash
python3 test_kit.py ALL
```

Submit your completed vasim.c to gradescope in the usual way.

## Note on FIFO replacement

In reai systems, the FIFO policy is modified with an LRU list and other steps to choose the coldest
page as the next victim---which might not be the one with oldest (first) mapping.   To do that, the OS must 
track enough references to approximate how warm or cold each page is, e.g., by maintaining a list in
approximate LRU order, as in the KVcache project.

In virtual memory, once a page is mapped, subsequent accesses to it are completed by hardware with no software 
intervention.   That means the OS needs some tricks to find out about which pages are in active use.
The tricks to do that are an OS topic called Two-Handed Clock Algorithm or FIFO with Second Chance.
in contrast, every access in KVcache is handled in software, so it is easy to maintain an LRU list.
