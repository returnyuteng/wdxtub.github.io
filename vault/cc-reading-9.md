title: 云计算 阅读材料 9 Resource Virtualization - CPU
categories:
- Technique
toc: true
date: 2016-01-24 11:37:56
tags:
- CMU
- 云计算
- 讲义
---

A key element, which is also one of the more complex parts of virtualization, is CPU virtualization.

<!-- more -->

---

Virtualizing a CPU entails two major steps:

+ Multiplexing a physical CPU (pCPU) among virtual CPUs (vCPUs) associated with virtual machines (VMs); this is called vCPU scheduling.
+ Virtualizing the ISA of a pCPU in order to make vCPUs with different ISAs run on this pCPU.

In this module, we present some conditions for virtualizing ISAs. Second, we describe ISA virtualization. Third, we make a distinction between two types of ISA virtualization: full virtualization and paravirtualization. Fourth, we discuss emulation, a major technique for virtualizing CPUs. Fifth, we compare two kinds of VMs: simultaneous multiprocessing (SMP) and uniprocessors (UP) VMs. Finally, we close with a discussion on vCPU scheduling. As examples, we present two popular Xen vCPU schedulers.

## The Conditions for Virtualizing ISAs

**Virtualizing an ISA**

The key to virtualizing a CPU lies in the execution of both privileged and unprivileged instructions issued by guest virtual processors. The set of any processor instructions is documented and provided in the ISA. The following video (Video 3.5) outlines the issues regarding virtualizing an ISA:

[Video 3.5: Virtualizing an ISA.](http://youtu.be/L55ut9i10O8)

Special privileges to system resources are permitted by defining modes of operations (or rings) in the ISA. Each CPU ISA usually specifies two modes of operations, system (or supervisor/kernel/privileged) mode and user mode (see Fig. 3.18(a)). System mode allows a wide accessibility to system components, while user mode restricts such accessibility. In an attempt to provide security and resource isolations, OSs in traditional systems are executed in system mode, while associated applications are run in user mode. Some ISAs, however, support more than two rings. For instance, the Intel IA-32 ISA supports four rings (see Fig. 3.18(b)). In traditional systems, when Linux is implemented on an IA-32 ISA, the OS is executed in Ring 0 and application processes are executed in Ring 3.

![Figure 3.18: System modes of operations (or rings).](/images/14551131967470.jpg)

A privileged instruction is defined as one that traps in user mode and does not trap in system mode. A trap is a transfer of control to system mode wherein the hypervisor (as in virtualization) or the OS (as in traditional OSs) performs some action before switching control back to the originating process. Traps occur as side effects of executing instructions. Overall, instructions can be classified into two different categories: sensitive and innocuous. Sensitive instructions can be either control sensitive or behavior sensitive. Control-sensitive instructions are those that attempt to modify the configuration of resources in a system, such as changing the mode of operation or CPU timer. An example of control-sensitive instructions is load processor status word (LPSW) (IBM System/370). LPSW loads the processor status word from a location in memory if the CPU is in system mode and, otherwise it traps. LPSW contains bits that determine the state of the CPU. For instance, one of these bits is the P bit, which specifies whether the CPU is in user mode or system mode. If executing this instruction is allowed in user mode, a malicious program can easily change the mode of operation to privileged and obtain control over the system. Hence, to protect the system, such an instruction can only be executed in system mode. Behavior-sensitive instructions are those whose behaviors are determined by the current configuration of resources in a system. An example of behavior-sensitive instructions is Pop Stack into Flags Register (POPF) (Intel IA-32). POPF pops the flag registers from a stack held in memory. One of these flags, called the interrupt enable flag, can be altered only in system mode. If POPF is executed in user mode by a program that attempts to pop the interrupt enable flag, POPF will act as a no op (i.e., no operation). Therefore, the behavior of POPF depends on the mode of operation, rendering as behavior sensitive. Finally, if the instruction is neither control sensitive nor behavior sensitive, it is innocuous.

According to Popek and Goldberg (1974), a hypervisor can be constructed if it satisfies three properties: efficiency, resource control, and equivalence. Efficiency entails executing all innocuous instructions directly on hardware without any interference by the hypervisor. Resource control suggests that it is not possible for any guest software to change the configuration of resources in a system. Equivalence requires identical behavior of a program running on a VM or on a traditional OS with no virtualization. One exception is a difference in performance. Popek and Goldberg's proposal (or theorem) implies that a hypervisor can be constructed only if the set of sensitive instructions is a subset of the set of privileged instructions. That is to say, instructions that interfere with the correct functioning of the system (i.e., sensitive instructions, such as LPSW) should always trap in the user mode. Figure 3.19 illustrates Popek and Goldberg's theorem. [1]

![Figure 3.19: Demonstrating Popek and Goldberg's theorem.](/images/14551132129213.jpg)

Finally, let us discuss how a trap can be handled in a system. Specifically, we describe traps in the context of CPU virtualization. Figure 3.20 demonstrates how a hypervisor can handle an instruction trap. The hypervisor's trap-handling functions can be divided into three main parts: dispatcher, allocator, and a set of interpreter routines. First, a privileged instruction traps to the hypervisor's dispatcher. If the hypervisor recognizes that the instruction is attempting to alter system resources, it directs it to the allocator; otherwise, it sends it to a corresponding interpreter routine. The allocator decides how system resources are to be allocated in a nonconflicting manner and satisfies the instruction's request accordingly. The interpreter routines emulate (more on emulation shortly) the effects of the instruction when operating on virtual resources. When the instruction is handled fully (i.e., done), control is passed back to the guest software at the instruction that comes immediately after the one that caused the trap.

![Figure 3.20: Demonstration of a trap to a hypervisor. The hypervisor includes three main components: the dispatcher, the allocator, and the interpreter routines.](/images/14551132303380.jpg)

**References**

1. Popek, J., and Goldberg, R. (1974). "Formal Requirements for Virtualizable Third Generation Architectures." Commun. ACM, Vol. 17, No. 7.

## Full Virtualization and Paravirtualization

A problem arises when an instruction that is sensitive but unprivileged is issued by a process running on a VM in user mode. According to Popek and Goldberg (1974), sensitive instructions have to trap to the hypervisor if executed in user mode. However, as explained earlier, sensitive instructions can be privileged (e.g., LPSW) and unprivileged (e.g., POPF). Unprivileged instructions do not trap to the hypervisor. Instructions that are sensitive and unprivileged are called critical (see Fig. 3.21). ISAs that contain critical instructions do not satisfy Popek and Goldberg's theorem [1] . Video 3.6 covers this concept and ways around it:

[Video 3.6: ISA virtualization techniques.](http://youtu.be/ay0ZpkMbw_8)

The challenge becomes, can a hypervisor be constructed in the presence of critical instructions? The answer is yes. Nonetheless, Smith and Nair [2] distinguish between a hypervisor that complies with Popek and Goldberg's theorem and one that does not comply by referring to the former as a true or an efficient hypervisor and to the latter simply as a hypervisor.

![Figure 3.21: Instructions that do not satisfy Popek and Goldberg’' theorem. They are called critical instructions.](/images/14551133049722.jpg)

If a processor does not satisfy Popek and Goldberg's virtualization requirement, a hypervisor can be constructed by using code patching, full virtualization, and/or paravirtualization. As illustrated in Figure 3.22, code patching requires the hypervisor to scan the guest code before execution, discover all critical instructions, and replace them with traps (system calls) to the hypervisor. Full virtualization emulates all instructions in the ISA. Emulation degrades performance because it implies reproducing the behavior of every source instruction by first translating it to a target instruction, then running it on a target ISA (more on emulation shortly). Paravirtualization deals with critical instructions by modifying guest OSs. Specifically, it entails rewriting every critical instruction as a hypercall that traps to the Xen hypervisor. Accordingly, paravirtualization improves performance due to totally avoiding emulation but at the expense of modifying guest OSs. In reverse, full virtualization avoids modifying guest OSs but at the expense of degrading system performance. As examples, VMware uses full virtualization, while Xen employs paravirtualization. Xen supports most major OSs, including Windows, Linux, Solaris, and NetBSD.

![Figure 3.22: Code scanning and patching to enforce critical instructions to trap to the hypervisor. The code is shown in a format close to a control flow diagram.](/images/14551133248973.jpg)

**References**

1. Popek, J., and Goldberg, R. (1974). "Formal Requirements for Virtualizable Third Generation Architectures." Commun. ACM, Vol. 17, No. 7.
2. Smith, J. E., and Nair, R. (2005). "The Architecture of Virtual Machines." Computer, 38(5), 32-38.

## Emulation

Now that we understand the conditions for virtualizing ISAs and the two main classes of CPU virtualization, full virtualization and paravirtualization, we move to discussing emulation as a technique used to implement full virtualization and process VMs. Emulation has been introduced previously, but to recap, emulation is the process of allowing the interfaces and functionalities of one system (the source) to be implemented on a system with different interfaces and functionalities (the target). Emulation is the only CPU virtualization mechanism available when the guest and host ISAs are different. If the guest and host ISAs are identical, direct native execution can (possibly) be applied.

Emulation is carried out either via interpretation or binary translation. With interpretation, source instructions are converted, one instruction at a time, to relevant target instructions. Interpretation is relatively slow because of the one-by-one emulation of instructions and the lack of any optimization technique (e.g., precluding the interpretation of an already encountered and interpreted instruction). Binary translation optimizes interpretation via converting blocks of source instructions to target instructions and caching generated blocks for repeated use. Typically, blocks of instructions are more amenable to optimizations than single instructions. Compared to interpretation, binary translation is much faster because of the application of block caching as well as code optimizations over blocks. In the following video, we discuss three major interpretation schemes: decode and dispatch, indirect threaded, and direct threaded.

[Video 3.7: Interpretation Techniques in Virtualization](http://youtu.be/TutPj22Dq94)

As explained in Video 3.7, a basic interpreter should read through the source code instruction by instruction, analyze each instruction, and call relevant routines to generate the target code. An interpreter called decode and dispatch applies basic interpretation but results in a number of branch (or jump) instructions, both direct and indirect, which leads to poor execution times. As an optimization for decode and dispatch, an interpreter called indirect threaded attempts to release some of the decode-and-dispatch branches via appending (or threading) a portion of the dispatch code to the end of each interpreter routine. Last, a more advanced interpreter, called direct threaded, improves indirect threaded by attempting to interpret a repeated operation only once. Although the direct-threaded interpreter improves the indirect threaded and the decode-and-dispatch branches, it still suffers from major drawbacks, such as vast memory image and limited portability. In the next video, we discuss binary translation, which essentially targets the limitations of direct-threaded interpretation.

[Video 3.8: Binary Translation.](http://youtu.be/wnbYwUm6KgY)

As presented in Video 3.8, binary translation tries to amortize the fetch and analysis costs caused by the direct-threaded interpreter through: (1) translating a block of source instructions (rather than a single instruction) to a block of target instructions and (2) caching the translated code in an attempt to save interpreting source instructions more than once. The following table compares binary translation, decode-and-dispatch, indirect-threaded, and direct-threaded emulation techniques in terms of four metrics: memory requirements, startup performance, steady-state performance, and code portability. For instance, the decode-and-dispatch interpreter row reads as follows: first, with decode and dispatch, memory requirements remain low because of having only one interpreter routine for each instruction type in the target ISA. Furthermore, decode and dispatch avoids threading the dispatch code at the end of each routine. Second, startup performance is fast because neither predecoding nor translation of the source binary is applied. Third, steady-state performance (i.e., the performance after starting up the interpreter) is slow because of the high number of branches and the interpretation of every instruction per each appearance. Finally, code portability is good because predecoding with addresses of interpreter routines is not applied by decode-and-dispatch interpreters (in contrast to direct-threaded interpreters).

![](/images/14551136108303.jpg)

## Virtual CPU

**Uniprocessor and Multiprocessor VMs**

As described earlier in the unit, a virtual CPU (vCPU) acts as a proxy to a physical CPU (pCPU). In other words, a vCPU is a representation of a pCPU to a guest OS (i.e., an OS that runs on a VM). A vCPU can be initiated in a VM and mapped to an underlying pCPU by the hypervisor. In principle, a VM can have one or many vCPUs. For instance, a VM in VMware ESX 4 can have up to eight vCPUs. The amount of vCPUs represents the width of a VM. A VM with a width greater than one is called a symmetric multiprocessing (SMP) VM. In reverse, a VM with a width equal to one is called a uniprocessor (UP) VM. Video 3.9 discusses Multiprocesseor VMs in detail:

[Video 3.9: Multiprocessor VMs.](http://youtu.be/0iZrCvccTdY)

Figure 3.23 demonstrates an SMP native system VM with a width of four and a UP native system VM, both running on the same hardware.

![Figure 3.23: An SMP native system VM with a width of four and a UP native system VM, both running on the same hardware.](/images/14551137270984.jpg)

Similar to a process on a general-purpose OS, a vCPU can be in different states, such as running, ready, and wait states. At a certain point in time, a vCPU can be scheduled by the hypervisor at only a single core (akin to scheduling an OS process on a core). For instance, a UP VM running on a host machine equipped with two Xeon 5405 (i.e., a total of eight pCPUs) will run only on one of the eight available cores. Inherently parallel workloads, such as MapReduce applications, prefer SMP VMs. We next discuss how the hypervisor schedules vCPUs on pCPUs.

**Virtual CPU Scheduling and Xen's Schedulers**

General-purpose OSs support two levels of scheduling, process and thread scheduling. With a hypervisor, one extra level of scheduling is added, vCPU scheduling. The hypervisor schedules vCPUs on the underlying pCPU(s), thereby providing each guest VM a portion of the underlying physical processing time.

We (briefly) discuss two popular schedulers from Xen, Simple Earliest Deadline First (SEDF) and Credit Scheduler (CS). As its name suggests, SEDF is simple, whereby only two parameters, n (the slice) and m (the period), are involved. A VM (or domain Ui in Xen's parlance) can request n every m. SEDF specifies a deadline for each vCPU computed in terms of n and m. The deadline is defined as the latest time a domain can be run to meet its deadline. For instance, a domain Ui can request n = 10ms and m = 100ms. Accordingly, a vCPU at this domain can be scheduled by SEDF as late as 90ms into the 100ms period, yet still meet its deadline. SEDF operates by searching across the set of all runnable vCPUs, held in a queue, and selecting the one with the earliest deadline.

Xen's Credit Scheduler (CS) is more involved than SEDF. First, when configuring a VM, each vCPU in the VM is set with two properties: the weight and the cap. The weight determines the share of a pCPU's capacity that should be provided to a vCPU. For instance, if two vCPUs, vCPU-1 and vCPU-2, are specified with weights of 256 (the default) and 128, respectively, vCPU-1 will obtain double the share of vCPU-2. Weights can range from 1 to 65,535. The cap determines the total percentage of a pCPU that should be given to a vCPU. The cap can modify the behavior of the weight. Nonetheless, a vCPU can be kept uncapped.

CS converts each vCPU's weight to credits. Credits from a vCPU will be deducted as long as it is running. A vCPU is marked as over once it runs out of credits and is otherwise marked under. CS maintains a queue per each pCPU (assuming a chip multiprocessors architecture) and stores all under vCPUs first, followed by all over vCPUs. CS operates by picking the first under vCPU in the queue to go next. CS keeps track of each vCPU's credits, and when switching a vCPU out, it places it in the queue at the end of the appropriate category. Finally, CS applies load balancing by allowing a pCPU with no under vCPUs to pull under vCPUs from queues of other pCPUs. More details on SEDF and CS can be found in Chisnall's The Definitive Guide to the Xen Hypervisor. [1]

Xen is an open source hypervisor. Hence, it is possible to devise and add your own scheduler. Chisnall provides an excellent and comprehensive coverage of Xen's internals as well as step-by-step instructions for adding a new scheduler to Xen.

**References**

1. David Chisnall (2007). "The Definitive Guide to the Xen Hypervisor." Prentice Hall.

## Summary

+ Virtualizing a physical CPU (pCPU) involves: (1) timesharing the pCPU among virtual CPUs (vCPUs) contained and executed in VMs (called vCPU scheduling) and (2) virtualizing the ISA of the pCPU to make it amenable to host vCPUs with different ISAs.
+ A vCPU acts as a proxy to a pCPU.
+ In principle, a VM can have one or many vCPUs.
+ A VM that includes more than one vCPU is called a symmetric multiprocessing (SMP) VM, while a VM with one vCPU is called a uniprocessor (UP) VM.
+ In principle, hypervisors can support three levels of scheduling, process, thread, and vCPU scheduling.
+ Examples of vCPU schedulers are Simple Earliest Deadline First (SEDF) and Credit Scheduler (CS) from Xen.
+ In addition to vCPU scheduling, virtualizing a pCPU requires virtualizing the instructions defined in its ISA.
+ Instructions in ISAs can generally be classified into two types: privileged and unprivileged instructions.
+ A privileged instruction is defined as one that traps in user mode and does not trap in system mode.
+ In addition, instructions can be further classified into two different categories: sensitive and innocuous.
+ Sensitive instructions can be either control sensitive or behavior sensitive.
+ Control-sensitive instructions are those that attempt to modify the configuration of resources in a system (e.g., LPSW from IBM System/370).
+ Behavior-sensitive instructions are those whose behaviors are determined by the current configuration of resources in a system (e.g., POPF from Intel IA-32).
+ When the instruction is neither control sensitive nor behavior sensitive, it is innocuous.
+ Sensitive instructions can be privileged (e.g., LPSW) and unprivileged (e.g., POPF).
+ Popek and Goldberg (1974) suggested that a hypervisor can only be constructed if the set of sensitive instructions is a subset of the set of privileged instructions (i.e., sensitive instructions always trap in the user mode).
+ A problem arises when instructions that are sensitive but unprivileged are issued in VMs running in user mode (i.e., they will not trap as such).
+ The instructions that are sensitive and unprivileged are called critical instructions.
+ A hypervisor can still be constructed for ISAs that contain critical instructions.
+ Constructing a hypervisor with the presence of critical instructions can be achieved using either code patching, full virtualization, and/or paravirtualization.
+ Code patching replaces all critical instructions with system calls to the hypervisor, thus enforcing them to trap.
+ Full virtualization emulates all instructions in the ISA.
+ Emulation is a popular technique in virtualizing CPUs. It allows the interfaces and functionalities of one system (the source) to be implemented on a system with different interfaces and functionalities (the target).
+ Emulation can be implemented using either interpretation or binary translation.
+ Interpretation techniques (e.g., decode and dispatch, indirect threaded, and direct threaded) translate source instructions to target instructions one at a time, while binary translation converts blocks of source instructions to target instructions and caches them for repeated use.
+ Paravirtualization rewrites every critical instruction as a hypercall that traps to the hypervisor (which typically requires modifying guest OSs).
+ As concrete examples, VMware uses full virtualization, while Xen employs paravirtualization.


