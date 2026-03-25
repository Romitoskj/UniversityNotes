# Virtualization

## 1. Core Characteristics of Virtualization

Virtualization environments provide three main benefits:

- **Increased Security:** The Virtual Machine Manager (VMM) controls and filters all guest activity, preventing the execution of harmful operations. It hides the host's physical resources from the guest, providing isolation and protecting sensitive information without needing complex security policies. _Examples: Java Virtual Machine (JVM) sandboxing, or hardware virtualization completely separating the guest's file system from the host's_.

- **Managed Execution:** Virtualization allows for flexible resource management, such as sharing a single physical machine among multiple virtual ones completely separating them (isolation), or aggregating distributed resources to look like a single centralized system (e.g., Cassandra distributed data store). It also allows for the emulation of older processors so legacy software can continue to run on modern hardware.
	
	![697](Images/Pasted%20image%2020260325221953.png)

- **Portability:** The ability to easily transfer and use data/applications across different computing platforms. Just as a Java application runs anywhere with a JVM, a Virtual Machine (VM) can be easily migrated because it boots from a disk image file (e.g., .vmdk, .vdi, .ami) that can be moved or converted to different formats.

## 2. Machine Reference Model & Instructions

To understand how virtualization techniques work, it is essential to understand the underlying computer architecture, specifically the Instruction Set Architecture (ISA).

![](Images/Pasted%20image%2020260325223359.png)

Instructions are divided into distinct categories:

- **Nonprivileged instructions:** Instructions that can be used safely without interfering with other tasks (e.g., floating-point or arithmetic instructions). These typically run in user mode.
- **Privileged instructions:** Sensitive operations executed under specific restrictions, typically requiring kernel/supervisor mode.
    - _Behavior-sensitive instructions:_ Operate on I/O devices or memory.
    - _Control-sensitive instructions:_ Alter the state of the CPU registers.

![](Images/Pasted%20image%2020260326003506.png)

## 3. The Hypervisor / Virtual Machine Manager (VMM)

The VMM (or hypervisor) sits above the hardware to mediate between the user mode and the supervisor (kernel) mode, typically running directly in supervisor mode (Ring 0).

According to Popek and Goldberg (1974), a VMM must satisfy three fundamental properties:

1. **Equivalence:** A guest OS must behave exactly as it would if it were executing directly on the physical host.
2. **Resource Control:** The VMM must be in complete control of all virtualized system resources.
3. **Efficiency:** A statistically dominant fraction of machine instructions must be executed directly on the hardware without the VMM's intervention.

**Types of Hypervisors:**

- **Type 1 (Native/Bare Metal):** Runs directly on the host's hardware. They can be built as a _Microkernel_ (e.g., Xen, Microsoft Hyper-V), which handles memory and CPU but leaves device drivers to the host OS, or a _Monolithic_ architecture (e.g., VMware ESX), which includes all drivers internally.
- **Type 2 (Hosted):** Runs as an application on top of a conventional host operating system (e.g., VMware Workstation, VirtualBox).

![697](Images/Pasted%20image%2020260325224759.png)

## 4. Virtualization Techniques and Challenges

To handle sensitive instructions, hypervisors use different techniques:

### A. Full Virtualization (using Binary Translation)

>[!info] The Trap Mechanism 
>- A "trap" is a hardware mechanism used by the Virtual Machine Manager (VMM) or hypervisor to intercept and take control of sensitive operations attempted by a guest Operating System (in user mode).
>- When a guest OS or user program attempts to execute a privileged instruction that is only allowed in kernel mode, the hardware generates a trap, transferring control directly to the hypervisor.

- The guest OS is completely unaware that it is being virtualized.

- The VMM scans the instruction stream. Noncritical instructions run directly on the hardware.

- When a sensitive instruction is detected by an hardware trap, the VMM replaces the sensitive instructions with a call to a hypervisor procedure that safely emulates the hardware's behavior using binary translation.

- **Binary Translation**: is a specific technique in which the VMM translates the binary instruction code of a VM into instructions that the underlying hardware can understand. This process allows for the emulation of the behavior of critical instructions without requiring modifications to the guest operating system. The hypervisor rewrites the guest's code one "basic block" at a time prior to execution. A basic block is a short sequence of instructions ending with a branch.
	
	![470](Images/Pasted%20image%2020260325235533.png)
	
- This is highly efficient because translated blocks are cached for future use, and user-level instructions (that are the majority) do not require translation.

### B. Paravirtualization

- Unlike full virtualization, the guest OS is explicitly modified to be aware that it is running inside a VM.
- An intelligent compiler replaces non-virtualizable or sensitive OS instructions with direct API calls to the hypervisor, known as **hypercalls**.
- This approach avoids the massive performance overhead caused by continuous hardware traps.
- Paravirtualization is much easier to implement on systems utilizing a Microkernel architecture, as fewer OS modules need to be modified. Examples include Xen and KVM.

![](Images/Pasted%20image%2020260325234345.png)

### C. Hardware-assisted Virtualization

**Hardware-assisted virtualization** is a technology introduced by hardware manufacturers, such as Intel (with VT-x) and AMD (with AMD-V), to natively support and improve the performance of virtualized environments directly at the processor level.

To understand its value, it is essential to look at the historical challenge it solved. Early x86 architectures were incredibly difficult to virtualize because 17 sensitive instructions could be executed in user mode without generating a hardware trap to alert the hypervisor. This broke the isolation between guest operating systems and forced hypervisors to rely on heavy, software-based "binary translation"—a process of constantly scanning and rewriting the guest's code on the fly to intercept these uncooperative instructions.

Hardware-assisted virtualization solved this by completely redesigning the processor's instruction set architecture. Its key mechanisms and benefits include:

- **Sensitive Instructions Become Privileged:** The redesigned architecture ensures that all sensitive instructions effectively behave as privileged instructions. Consequently, if a guest operating system attempts to execute a sensitive instruction, the hardware guarantees that a trap is successfully generated, transferring control safely to the hypervisor.
- **Direct Hardware Execution:** Because the hardware now natively guarantees that sensitive operations will be trapped, both privileged and non-privileged instructions can run directly on the bare-metal hardware.
- **Reduced Performance Penalties:** By eliminating the need for continuous binary translation, the system avoids severe performance overhead. Furthermore, because only specific operations (like I/O instructions) generate traps, the total volume of traps the hypervisor must process is significantly reduced.