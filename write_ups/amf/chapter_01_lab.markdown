---
layout: post
# title: "AMF CH 01"
date: 2022-01-07 14:07 -500
categories: jekyll update
---

# Art of Memory Forensic Chapter 1

Question 1: What component assists the CPU in address translation?
- [x] The Memory Management Unit (MMU)
- [ ] The Address Translation Unit (ATU)
- [ ] The Central Memory Hub (CMH)
- [ ] The Memory Management Controller (MMC)

The memory management unit in a computer is responsible for translating virtual memory addresses to physical addresses. (Found in the `CPU and MMU` section)  

Question 2: When dealing with raw, padded memory dumps, a physical address is an offset into the memory dump file.
- [x] True
- [ ] False

Since we are dealing with the virtual memory of a system, the physical addresses can be calculated using the information gathered from a memory dump. This physical address location could be considered an offset in the memory dump file. (Found in the note under `Address Space`)  

Question 3: Which statement(s) are false?
- [ ] IA32 Architecture is also known as x86
- [ ] Physical Address Extension (PAE) allows up to 64GB of physical memory
- [x] 64-bit CPUs only actuall use 52 bits of the available address space
- [ ] A typical page size is 4KB, but it can be larger if the pate size entry (PSE) flag is set
- [ ] All of the above

A 64-bit CPU uses 48 bits of the available address space. Because of this, virtual addresses on these systems are in canonical format, meaning that bits 63:48 are either all 1's or all 0's depending on bit 47. (`Intel 64 Architecture` section of the chapter)  

Question 4: Which CPU register is used to store the directory table base (page directory base)?
- [ ] CR0
- [ ] EAX
- [x] CR3
- [ ] DR3

CR3 contains the physical address of the initial structure used for address translation. During context switches, it is updated when a new task is scheduled. (`Registers` section of the chapter)  

Question 5: Which statement(s) are true?
- [ ] Paging allows processes to "see" more RAM than is physically present
- [ ] The page fault handler code must never be paged
- [ ] Paging complicates memory forencis because not all data is memory resident at the time of acquisition
- [ ] Paging writes potentially valuable volatile evidence to non-volatile storage such as disk
- [x] All of the above

These ideas are discussed in the `Demand Paging` section of the chapter.

Question 6: The winlogon.exe process (PID 628) in sample001.bin has a virtual address 0x77a80000 and DTB value 0x682e000. What is the corresponding physical offset? What do you see at the physical offset within the file?  

First convert the virtual address to binary:   
31 | 0111 0111 1010 1000 0000 0000 0000 0000 | 0

```
==============================================
|Paging Structure     | Binary       | Hex   |
|--------------------------------------------|
|Page Directory Index | 0111011110   | 0x1DE |
|Page Table Index     | 1010000000   | 0x280 |
|Address Offset       | 000000000000 | 0x000 |
==============================================
```

ad3d de09

10101101.00111101.11011110.00001001
173.61.222.9