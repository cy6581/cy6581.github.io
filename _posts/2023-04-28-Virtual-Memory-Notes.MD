---
layout: post
title: My Attempt To Explain Computer Memory
date: 2023-05-03 15:07:12 +0800
categories: ["Memory", "JavaScript"]
---

Being a predominantly JavaScript developer, I never had to worry much about memory management because the runtime would do it for me. This all changed when I encountered an NPM package that wasn't actually written in JavaScript, but in Rust, and required you to call an exposed .free() API to prevent memory leaks. That is another story, but it did lead me down this rabbit hole to understand what the whole fuss about memory was about.

What follows is an incomplete, but hopefully clear, nuts-and-bolts explanation of what virtual memory is, and how computer programs use it:

- Applications don't actually interact with the Physical Memory on your machine. Instead, they see an abstraction known as Virtual Memory, where they can store data at memory locations known as Virtual Addresses.

- The size of the Virtual Address space typically corresponds to the size of the CPU registers on your processors. For instance, an x86 processor (aka a 32-bit CPU) can handle memory addresses from 0 to 2 ^ 32 - 1. Meanwhile, an x64 processor (aka 64-bit CPU), can handle memory addresses from 0 to 2 ^ 64 - 1. The minus one happens because we count from zero.

- Typically, each memory address is a storage for one BYTE (8 bits) of data. Larger pieces of data will be stored over several addresses. For example, a 32-bit integer (i32 in many languages) will be stored over 4 memory addresses, with each address storing 1/4 of the integer.

- Every application (i.e. every process) has its own personal memory address space, barring security flaws. Which means it sees 2 ^ 64 available virtual memory addresses on an x64 machine. If you have 10 running processes, each process can storage different pieces of data at their respective 0 memory address, and not collide with each other. How is that achieved?

- Every virtual address, for every process, is actually mapped to a different location on physical memory. This mapping is stored in a structure known as a Page Table. Every process has its own Page Table. But what are Pages?

- More concretely, virtual memory is divided into units known as Pages (typically 4KB in older machines). They are contiguous sections of memory, which means that the memory address is in running order. Every Page is Virtual Memory maps 1:1 to a Page Frame in Physical Memory in the Page table. When applications request memory, they allocated in the form of Pages, which is "backed up" by a Page Frame in the physical hardware. Your OS will try to keep Page Frames continguous too, but sometimes this is not possible.

- There can be "wastage" for page sizes that are too big, as even if an application requires less than 4KB, for example, it is given a page of 4KB memory. If the page size is too small, the Page Table will have to keep track of many many many more entries, which makes the mapping cumbersome. Here we observe a trade off in determining the optimal page size for machines. In recent computers, these are usually around 1MB in size per page.

- Where do page tables live? Preferrably, they live in the processor, so that access is fast. In reality, they are often too large and need to sit in memory. This will mean that look ups and slower, and you might notice that to access memory, we now have a pre-processing step that itself involves lookup in memory.

- Page Tables can be either Flat or Multi Level. The problem with Flat page tables comes when the address space is large (e.g. 64-bit memory addresses). The page table will then become humongous,
  because they need to set aside contingous amount of space for ALL the page table entries, even those which are unused. And there are many that are unused, as applications typically do not address all the memory that is available to them. **Note the paradox arising from Flat page tables: we could end up using MORE MEMORY to perform memory address translation, than the memory used by actual programs.**

- The Multi Level Page table solves this issue. The concept is to split the virtual addresses into chunks. The First level Page table stores only the possible entries for the first chunk, e.g. (if the first chunk is 2-bits long, it stores only 4 entries). The entries point to the memory location of the respective Second level page table, which maps the possible values in the second chunk of the address. Entries in the Second level page table then point to location the Third level page table, and so on.

- Where do the space savings happen? **The concept is that programs usually address memory in sequence, clustered at both ends of the address space, beginning and end, for the heap and stack**. This means that only the first or last entries in the First level table point to something. In the Second level, similarly, it is likely only one or two entries point to actual next level tables. In this way, we actually save out on space for for a large portion of the memory addresses, by using very few page tables in total.

- The slowest part of using Multi Level page tables is in finding the next level page table, which as discussed, often resides in memory. In the worst case, for a 4 level page table, you might need up to 4 memory accesses.

- The trick to speeding up address translation is a **piece of hardware in the processor called the Translation Look-aside Buffer (TLB)**. This caches the entire phyiscal address from previous translations so you don't have to traverse the hierarchy of multi level page tables every time. Due to the fact the programs often access the same memory sites multiple times in a function call, it is very likely for queries to result in a TLB hit.

- To optimise the TLB further, there can be multiple TLBs at different levels. The smallest level holds fewer entries, but reads are blazing fast. The second level holds more (e.g. few thousand) entries, and the reads are slightly slower, but way faster than memory access. **This concept is slightly different from Multi Level page tables, but borrowing the similar idea that you don't have to keep everything in one place.** You can keep the most frequently used things in a separate place from the others.

- In higher-end processors (e.g., non-embedded systems), the Virtual-to-Physical Address translation, and updating of the TLB is done at the hardware level. This leads to very fast memory address translation.
