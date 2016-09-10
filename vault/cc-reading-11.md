title: 云计算 阅读材料 11 Resource Virtualization - I/O
categories:
- Technique
toc: true
date: 2016-01-26 11:37:56
tags:
- CMU
- 云计算
- 讲义
---

Our final segment in resource virtualization is that of IO devices. In this respect, we can consider the VMM or hypervisor the arbiter of communication between multiple guests and the physical hardware, multiplexing the usage (in time/space or both), depending on the actual device being shared.

<!-- more -->

---

The virtualization strategy for a given I/O device type consists of:

1. Constructing a virtual version of that device, and
2. Virtualizing the I/O activity routed to the device.

Typical I/O devices include disks, network cards, displays, and keyboards. As discussed previously, the hypervisor might create a virtual display as a window on a physical display. In addition, a virtual disk can be created by assigning to it a fraction of the physical disk's storage capacity. After constructing virtual devices, the hypervisor ensures that each I/O operation is carried out within the bounds of the requested virtual device. For instance, if a virtual disk is allocated 100 cylinders from among 1,000 cylinders provided by a physical disk, the hypervisor guarantees that no I/O request intended for that virtual disk can access any cylinder other than the 100 assigned to it. More precisely, the disk location in the issued I/O request will be mapped by the hypervisor to only the area where the virtual disk has been allocated on the physical disk. Next, we cover some I/O basics and then move on to the details of I/O virtualization. After covering the basics, we will examine the case of Xen, and how it handles IO virtualization.

## I/O Basics

**Basics of I/O**

To begin, each I/O device has a device controller. A device controller can typically be signaled either by a privileged I/O instruction or memory-mapped I/O. I/O instructions are provided by ISAs. Intel-32 is an example of a processor that provides I/O instructions in its ISA. Many recent processors, however, allow performing I/O between the CPU and the device controllers through memory-mapped I/O (e.g., RISC processors). As shown in Figure 3.28, with memory-mapped I/O, a specific region of the physical memory address space is reserved for accessing I/O devices. These addresses are recognized by the memory controller as commands to I/O devices and do not correspond to memory physical locations. Different memory-mapped addresses are used for different I/O devices. Finally, in order to protect I/O devices, both I/O instructions and memory-mapped addresses are handled in system mode, thus becoming privileged.

![Figure 3.28: Memory mapped I/O with a specific region in the RAM address space for accessing I/O devices.](/images/14556004831082.jpg)

Because I/O operations are executed in system mode, user programs can only invoke them through OS system calls (assuming traditional systems). The OS abstracts most of the details of I/O devices and makes them accessible through only well-defined interfaces. Figure 3.29 shows the three major interfaces that come into play when a user program places an I/O request. These are the system call interface, the device driver interface, and the operation-level interface. Starting an I/O operation, a user I/O request causes an OS system call that transfers control to the OS. Next, the OS calls device drivers (a set of software routines) via the device driver interface. A relevant device driver routine converts the I/O request to an operation specific to the requested physical device. The converted operation is subsequently carried through the operation-level interface to the corresponding physical device.

![Figure 3.29: The three major interfaces involved in I/O operations: system call, device driver, and operation-level interfaces.](/images/14556005015461.jpg)

## Virtualizing I/O Devices

**I/O Virtualization**

I/O virtualization allows a single physical I/O device to be shared by more than one guest OS. Figure 3.30 demonstrates multiple guest OSs in native system VMs sharing a single hardware machine. As shown, the hypervisor constructs virtual devices from physical devices. A main observation is that both the guest OSs and the hypervisor must have device drivers encapsulating the interfaces to the devices. This means that with virtualization, two different device drivers must be supported per each device versus only one without virtualization. In reality, this is a problem because vendors of devices usually supply drivers for only the major OSs but not for hypervisors (though this could change in the near future). One way to circumvent such a problem is to collocate the hypervisor with a major OS (e.g., Linux) on the same machine. This way, I/O requests can be handled by the OS which holds all requisite I/O drivers. This is the approach adopted by Xen and discussed on the next page.

![Figure 3.30: Logical locations of device drivers in multiple guest OSs in native system VMs sharing a single hardware machine.](/images/14556006679296.jpg)

Moreover, with I/O virtualization, every I/O request issued by a user program at a guest VM should be intercepted by the hypervisor because I/O requests are all privileged and thus need to be controlled by the hypervisor. Clearly, this would entail a trap to the guest OS for every I/O request. All I/O requests are privileged, whether issued using I/O instructions or memory-mapped I/O; hence, they are not critical instructions, and they all trap to the hypervisor. As such, the hypervisor can easily intercept every I/O request simply when trapping. In principle, the hypervisor can intercept I/O requests at any of the three interfaces: the system call interface, the device driver interface, or the operation-level interface.

If the hypervisor intercepts an I/O request at the operation-level interface, some essential information about the I/O action might be lost. The hypervisor needs that information to handle I/O requests correctly. When an I/O request arrives at the device driver interface, it might get transformed into a sequence of instructions. When the sequence of instructions is received at the operation-level interface, it becomes difficult for the hypervisor to identify them as instructions for a single I/O request. For example, a disk write becomes multiple store instructions in case of memory-mapped I/O or multiple ISA I/O instructions. Hence, intercepting I/O requests at the operation-level interface typically is avoided. In contrast, intercepting an I/O request at the device driver interface allows the hypervisor to efficiently map the request to the respective physical device and transmit it through the operation-level interface. Clearly, this process is a natural point for I/O virtualization; yet it would oblige hypervisor developers to learn about the different device driver interfaces of various guest OSs in order to be able to intercept I/O requests. Last, intercepting I/O requests at the system call interface (i.e., the application binary interface [ABI]) might theoretically make the I/O virtualization process easier, whereby the entire I/O operation could be handled for each request by the hypervisor (the solo controller in this case). To achieve that goal, however, the hypervisor has to emulate the ABI routines of every guest OS (different OSs have different ABI routines). Consequently, hypervisor developers need also to learn about the internals of every potential guest OS. Furthermore, emulating ABI routines can degrade system performance due to the overhead imposed by the emulation process. In practice, intercepting I/O requests at the device driver interface can be more efficient. In the next video, we discuss the overall network virtualization process as applied to a physical network adapter.

[Video 3.12: Shared Devices in Virtualization. ](http://youtu.be/tMOAx08Ws3w)

As explained in Video 3.12, one physical adapter card can appear as multiple virtual network interface cards (vNICs), each with a separate MAC address and on the same network as the physical one. To network infrastructures, such as LANs and SANs, vNICs appear as regular physical cards.

## Xen's Approach to I/O Virtualization

**I/O Virtualization in Xen**

As a concrete example, we discuss Xen's approach to I/O virtualization. As we pointed out earlier, to get around the problem of having device drivers for the hypervisor as well as the guest OSs, Xen collocates its hypervisor with a traditional general-purpose OS. Figure 3.31 shows a host OS and the Xen hypervisor executing in full privileges at ring 0. Guest OSs run unprivileged at ring 1, while all processes at all domains (i.e., virtual machines) run unprivileged at ring 3. Clearly, the figure assumes a system with four rings (e.g., Intel-32). On systems with only two levels of privileges, the hypervisor and the host OS can execute in system mode, while domains and processes can execute in user mode. As illustrated in the figure, Xen eliminates the device drivers entirely from guest OSs and provides a direct communication between guest OSs at domain U and the host OS at domain 0. More precisely, every domain Ui in Xen will not hold any virtual I/O devices or relevant drivers. Rather, every I/O request is now transferred directly to domain 0, which by default hosts all the required device drivers necessary to satisfy all I/O requests. For instance, rather than using a device driver to control a virtual network card interface (vNIC), with Xen network, frames/packets are transferred through event channels directly to and from domain 0. This is done using NIC frontend and backend interfaces at domain Uj (in which j > 0) and U0, respectively. Likewise, no virtual disk is exposed to any guest OS, and all disk data blocks imposed by file reads and writes are delegated by Xen to domain 0.

![Figure 3.31: Xen's approach to I/O virtualization assuming a system with four rings (e.g., Intel-32). Xen collocates an OS, at a VM0 called domain 0, with the hypervisor on the physical platform to "borrow" its device drivers and avoid coding them in the hypervisor. This makes the hypervisor "thinner" and accordingly more reliable. Also, it makes it easier on the hypervisor developers.](/images/14556007538437.jpg)

## Resource Virtualization: I/O Summary

+ To virtualize an I/O device, we ought to follow two main steps: (1) construct a virtual version of the device and (2) virtualize the I/O activity routed to the device.
+ Constructing a virtual version of an I/O device entails sharing the device across multiple guest OSs.
+ Virtualizing the I/O activity to an I/O device passes through the device's controller (each I/O device has a device controller).
+ A device controller can be signaled via either a privileged I/O instruction (defined in ISA) or memory-mapped I/O.
+ I/O instructions and memory-mapped addresses are handled in system mode to protect the called I/O devices.
+ I/O instructions and memory-mapped addresses are not critical, thus can be easily handled by the hypervisor after naturally trapping to it (i.e., because they are privileged and not critical, they will naturally trap to the hypervisor when run in user mode).
+ General-purpose OSs abstract most of the details of I/O devices and make them accessible only through well-defined interfaces, such as the system call interface, the device driver interface, and the operation-level interface.
+ In the presence of a hypervisor, two different device drivers must be supported per each I/O device versus only one in traditional nonvirtualized systems.
+ The redundancy of device drivers in the presence of a hypervisor is usually circumvented by collocating the hypervisor with a major OS (e.g., Linux) on the same machine. Subsequently, the hypervisor leverages the device drivers of the major OS without requiring special device drivers (Xen applies this approach).
+ Because all I/O instructions are privileged, they need to be intercepted by the hypervisor.
+ In principle, the hypervisor can intercept I/O requests at any of the three interfaces: the system call interface, the device driver interface, and the operation-level interface.
+ Intercepting I/O requests at the operation-level interface might lead to the loss of some essential information about I/O actions.
+ Intercepting I/O requests at the system call interface (i.e., the ABI) entails emulating the ABI routines of every guest OS (different OSs have different ABI routines).
+ In practice, intercepting I/O requests at the device driver interface is typically the most efficient approach because it avoids emulating the ABI routines of every guest OS and losing some necessary information about I/O actions.


