title: 云计算 阅读材料 10 Resource Virtualization - Memory
categories:
- Technique
toc: true
date: 2016-01-25 11:37:56
tags:
- CMU
- 云计算
- 讲义
---

The next resource we will examine, with respect to virtualization, is memory. Memory virtualization should ring a bell; specifically, it is very closely related to the operating systems concept of virtual memory! As such, we will begin our discussion by recalling virtual memory concepts and then discuss memory virtualization as an extension of these techniques. VMWare has pioneered some interesting and clever techniques in the realm of memory reclamation from Guest OSes, which will also be covered in this module.

<!-- more -->

---

## One-Level Page Mapping

**Virtual Memory and One-Level Page Mapping**

Virtual memory is a well-known virtualization technique supported in most general-purpose OSs. The basic idea of virtual memory is that each process is provided with its own virtual address space, broken up into chunks called virtual pages. A page is a contiguous range of addresses. As shown in Figure 3.24, virtual memory maps virtual pages to physical pages in what is called a page table. We call this one-level page mapping between two types of addresses: the virtual and the physical. Each process in the OS has its own page table. A main observation pertaining to page tables is that not all virtual pages of a process need to be mapped to respective physical pages in order for the process to execute. When a process references a page that exists in the physical memory (i.e., there is a mapping between the requested virtual page and the corresponding physical page), a page hit is attained. On a page hit, the hardware obtains the required virtual to physical mapping with no further actions. In contrary, when a process references a page that does not exist in the physical memory, a page miss is incurred. On a page miss, the OS is alerted to handle the miss. Subsequently, the OS fetches the missed page from disk storage and updates the relevant entry in the page table.

![Figure 3.24: Mapping a process' virtual address space to physical address space. This is captured in what is called a page table. Each process has its own page table.](/images/14556001462899.jpg)

## Two-Level Page Mapping

Contrary to OSs in traditional systems, with system virtualization, the hypervisor allocates a contiguous addressable memory space for each created VM (not process). This memory space per a VM is called real memory. In return, each guest OS running in a VM allocates a contiguous, addressable memory space for each process in its real memory. This memory space per a process is called virtual memory (same name as in traditional systems). Each guest OS maps the virtual memories of its processes to the real memory of the underlying VM, while the hypervisor maps the real memories of its VMs to the system physical memory. Clearly, in contrast to traditional OSs, this entails two levels of mappings between three types of addresses: virtual, real, and physical. In fact, these virtual-to-real and real-to-physical mappings define system memory virtualization. This basic idea of memory virtualization via two-level page mapping is summarized in Video 3.10:

[Video 3.10: Memory Virtualization.](http://youtu.be/Gl0Dw7G9V5U)

![Figure 3.25: Memory virtualization in a native system VM.](/images/14556002072284.jpg)

Similar to any general-purpose OS, a guest OS would still own its set of page tables. In addition, the hypervisor would own another set of page tables for mapping real-to-physical addresses. The page tables in the hypervisor are called real map tables. Figure 3.25 demonstrates system memory virtualization in a native system VM. It shows page tables maintained by guest VMs and real map tables maintained by the hypervisor. Each entry in a page table maps a virtual page of a program to a real page in the respective VM. Likewise, each entry in a real map table maps a real page in a VM to a physical page in the physical memory. When a guest OS attempts to establish a valid mapping entry in its page table, it traps to the hypervisor. Subsequently, the hypervisor establishes a corresponding mapping in the relevant VM's real map table.

## Memory Overcommitment

In memory virtualization, the combined total size of real memories can grow beyond the actual size of physical memory. This concept is typically called memory overcommitment. Memory overcommitment ensures that physical memory is highly utilized by active, real memories (assuming multiple VMs running simultaneously). Indeed, without memory overcommitment, the hypervisor can only run VMs with a total size of real memories less than that of the physical memory. For instance, Figure 3.26 shows a hypervisor with 4GB of physical memory and three VMs, each with 2GB of real memory. Without memory overcommitment, the hypervisor can only run one VM because of not having enough physical memory to assign to two VMs at once. Although each VM would require only 2GB of memory, wherein the hypervisor has 4GB of physical memory, this memory cannot be afforded because hypervisors generally require overhead memories (e.g., to maintain various virtualization data structures).

![Figure 3.26: A hypervisor with 4GB of physical memory, enabling three VMs at once with a total of 6GB of real memory.](/images/14556002486593.jpg)

To this end, in practical situations, some VMs might be lightly loaded, while others might be heavily loaded. Lightly loaded VMs can cause some pages to sit idle, while heavily loaded VMs can result in memory page thrashing. To deal with such a situation, the hypervisor can take (or steal) the inactive physical memory pages away from idle VMs and provide them to heavily loaded VMs. As a side effect, hypervisors usually write zeros to the stolen/reclaimed, inactive physical memory pages in order to avert information leaking among VMs.

## Reclamation Techniques and VMware Memory Ballooning

**Memory Reclamation**

To maintain full isolation, guest OSs are kept unaware that they are running inside VMs. VMs are also kept unaware of the states of other VMs running on the same physical host. Furthermore, with multiple levels of page mapping, VMs remain oblivious of any physical memory shortage. Therefore, when the hypervisor runs multiple VMs at a physical host, and the physical memory turns stressed, none of the VMs can automatically help in freeing up memory.

The hypervisor deals with the situation by applying a reclamation technique. As its name suggests, a reclamation technique attempts to reclaim inactive real memory pages at VMs and make them available for the hypervisor when experiencing a memory shortage. Video 3.11 describes a couple of techniques which can be used to reclaim memory from guest operating systems:

[Video 3.11: Advanced Memory Management.](http://youtu.be/fAb6DeSNUv8)

One of the popular reclamation techniques is the ballooning process introduced in VMware ESX, which has been the basis for similar techniques in other hypervisors.

![Figure 3.27: The ballooning process in VMware ESX.](/images/14556003375691.jpg)

In VMware ESX, a balloon driver must be installed and enabled in each guest OS as a pseudo-device driver. The balloon driver regularly polls the hypervisor through a private channel to obtain a target balloon size. As illustrated in Figure 3.27, when the hypervisor experiences memory shortage, it inflates the balloon by setting a proper target balloon size. Figure 3.27(a) shows four real memory pages mapped to four physical pages, of which only two pages are actually active (the red and the yellow ones). Without involving the ballooning process, the hypervisor is unaware of the other two inactive pages (the green and the dark-blue ones) because they are still mapped to physical pages. Consequently, the hypervisor will not be able to reclaim inactive pages unless getting informed. With memory ballooning, however, the hypervisor can set the balloon target size to an integer number (say 2 or 3). When recognized by the balloon driver at the guest OS, the driver checks out the pages, locates the two inactive pages, and pins them (see Figure 3.27(b)). The pinning process is carried out by the guest OS via ensuring that the pinned pages cannot be read/written by any process during memory reclamation. After pinning the inactive pages, the balloon driver transmits to the hypervisor the addresses of the pinned pages. Subsequently, the hypervisor proceeds safely with reclaiming the respective physical pages and allocating them to needy VMs. Last, to unpin pinned pages, the hypervisor deflates the balloon by setting a smaller target balloon size and communicates that to the balloon driver. When received by the balloon driver, it unpins the pinned pages so the guest OS can utilize them. More information about the ballooning process can be found in VMware ESX's documentation.

## Resource Virtualization: Memory Summary

+ In the earliest days, either a process fit a memory or it could not be run.
+ Virtual memory changed the status quo by allowing a process that cannot fit a physical memory to run as if it essentially fits the memory.
+ An indirect inference of virtual memory is that multiple processes that cannot collectively fit a certain physical memory can now run altogether on this same physical memory.
+ The basic idea of virtual memory is that each process is provided with its own virtual address space.
+ The virtual address space of each process is translated to the physical address space that the physical memory uses.
+ The translation of virtual addresses to physical addresses is maintained in a software data structure called the page table.
+ In traditional systems (i.e., nonvirtualized environments), the virtual-to-physical translation is called one-level page mapping.
+ In virtualized environments (i.e., when a hypervisor is involved), the virtual-to-physical translation is extended at least one more level and called two-level page mapping.
+ The two-level page mapping entails two consecutive translations, virtual-to-real then real-to-physical translations. In this case, the real address space refers to the memory space of a VM, while the virtual and physical address spaces relate to the traditional memory spaces of processes and the physical memory.
+ As a result, memory virtualization in virtualized environments typically is perceived as an extension to the classic virtual memory concept supported in most general-purpose OSs.
+ When the combined total size of real memories grows beyond the actual size of the underlying physical memory, memory overcommitment is attained.
+ Memory overcommitment improves memory utilization via allowing VMs with aggregate real memories larger than the physical memory to run simultaneously.
+ Memory overcommitment, however, necessitates reclaiming inactive real memory pages at VMs and relocating them to the hypervisor when experiencing a physical memory shortage. This is called reclamation technique.
+ One of the popular reclamation techniques is the ballooning process incorporated in VMware ESX.


