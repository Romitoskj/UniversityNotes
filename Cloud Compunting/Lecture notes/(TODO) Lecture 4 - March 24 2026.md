# Virtualization

## 1. Core Characteristics of Virtualization

Virtualization environments provide three main benefits:

- **Increased Security:** The Virtual Machine Manager (VMM) controls and filters all guest activity, preventing the execution of harmful operations. It hides the host's physical resources from the guest, providing isolation and protecting sensitive information without needing complex security policies. _Examples: Java Virtual Machine (JVM) sandboxing, or hardware virtualization completely separating the guest's file system from the host's_.

- **Managed Execution:** Virtualization allows for flexible resource management, such as sharing a single physical machine among multiple virtual ones (isolation), or aggregating distributed resources to look like a single centralized system (e.g., Cassandra distributed data store). It also allows for the emulation of older processors so legacy software can continue to run on modern hardware.

	![](../../Pasted%20image%2020260325221953.png)

- **Portability:** The ability to easily transfer and use data/applications across different computing platforms. Just as a Java application runs anywhere with a JVM, a Virtual Machine (VM) can be easily migrated because it boots from a disk image file (e.g., .vmdk, .vdi, .ami) that can be moved or converted to different formats.

## 2. Machine Reference Model & Instructions

To understand how hypervisors work, it is essential to understand the underlying computer architecture, specifically the Instruction Set Architecture (ISA). Instructions are divided into distinct categories:

- **Nonprivileged instructions:** Instructions that can be used safely without interfering with other tasks (e.g., floating-point or arithmetic instructions). These typically run in user mode.
- **Privileged instructions:** Sensitive operations executed under specific restrictions, typically requiring kernel/supervisor mode.
    - _Behavior-sensitive instructions:_ Operate on I/O devices or memory.
    - _Control-sensitive instructions:_ Alter the state of the CPU registers.

## 3. The Hypervisor / Virtual Machine Manager (VMM)

The VMM (or hypervisor) sits above the hardware to mediate between the user mode and the supervisor (kernel) mode, typically running directly in supervisor mode (Ring 0).

According to Popek and Goldberg (1974), a VMM must satisfy three fundamental properties:

1. **Equivalence:** A guest OS must behave exactly as it would if it were executing directly on the physical bare-metal hardware.
2. **Resource Control:** The VMM must be in complete control of all virtualized system resources.
3. **Efficiency:** A statistically dominant fraction of machine instructions must be executed directly on the hardware without the VMM's intervention.

**Types of Hypervisors:**

- **Type 1 (Native/Bare Metal):** Runs directly on the host's hardware. They can be built as a _Microkernel_ (e.g., Xen, Microsoft Hyper-V), which handles memory and CPU but leaves device drivers to the host OS, or a _Monolithic_ architecture (e.g., VMware ESX), which includes all drivers internally.
- **Type 2 (Hosted):** Runs as an application on top of a conventional host operating system (e.g., VMware Workstation, VirtualBox).

## 4. Virtualization Techniques and Challenges

A major historical challenge was that the original x86 architecture was not natively virtualizable because some sensitive instructions could be executed in user mode without triggering a trap, breaking isolation. Hardware manufacturers solved this by introducing **Hardware-supported Virtualization (Intel VT-x and AMD-V)**, which redesigned the instruction set so that all sensitive instructions automatically generate a hardware trap to the hypervisor.

To handle sensitive instructions, hypervisors use different techniques:

### A. Full Virtualization (using Binary Translation)

- The guest OS is completely unaware that it is being virtualized.
- The VMM scans the instruction stream. Noncritical instructions run directly on the hardware.
- When a sensitive instruction is detected, the VMM uses **Binary Translation**: it rewrites the guest's code one "basic block" at a time prior to execution. A basic block is a short sequence of instructions ending with a branch. The VMM replaces the sensitive instructions with a call to a hypervisor procedure that safely emulates the hardware's behavior.
- This is highly efficient because translated blocks are cached for future use, and user-level processes do not require translation.

### B. Paravirtualization

- Unlike full virtualization, the guest OS is explicitly modified to be aware that it is running inside a VM.
- An intelligent compiler replaces non-virtualizable or sensitive OS instructions with direct API calls to the hypervisor, known as **hypercalls**.
- This approach avoids the massive performance overhead caused by continuous hardware traps.
- Paravirtualization is much easier to implement on systems utilizing a Microkernel architecture, as fewer OS modules need to be modified. Examples include Xen and KVM.