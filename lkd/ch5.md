### **Chapter 5. System Calls**

> Look at `ioctl()` as an example of what not to do (when implementing a system call).
> <small>Robert Love</small>

In any modern operating system, the kernel provides a set of interfaces by which processes running in user-space can interact with the system. These interfaces give applications: [p69]

* controlled access to hardware,
* a mechanism with which to create new processes and communicate with existing ones,
* and the capability to request other operating system resources.

The existence of these interfaces, and the fact that applications are not free to directly do whatever they want, is key to providing a stable system

### Communicating with the Kernel

System calls provide a layer between the hardware and user-space processes, which serves three primary purposes:

1. **Providing an abstracted hardware interface for userspace.**
    * For example, when reading or writing from a file, applications are not concerned with the type of disk, media, or even the type of filesystem on which the file resides.
2. **Ensuring system security and stability.** The kernel acts as a middleman between system resources and user-space, so it can arbitrate access based on permissions, users, and other criteria.
    * For example, this arbitration prevents applications from incorrectly using hardware, stealing other processes’ resources, or otherwise doing harm to the system.
3. **A single common layer between user-space and the rest of the system allows for the virtualized system provided to processes.**  It would be impossible to implement multitasking and virtual memory if applications were free to access access system resources without the kernel’s knowledge. [p69]

In Linux, system calls are the only means user-space has of interfacing with the kernel and the only legal entry point into the kernel other than exceptions and traps. Other interfaces, such as device files or `/proc`, are ultimately accessed via system calls. Interestingly, Linux implements far fewer system calls than most systems.

### APIs, POSIX, and the C Library

#### APIs *

Applications are typically programmed against an Application Programming Interface (API) implemented in user-space, not directly to system calls, because no direct correlation is needed between the interfaces used by applications and the actual interface provided by the kernel.

An API defines a set of programming interfaces used by applications. Those interfaces can be:

* implemented as a system call,
* implemented through multiple system calls, or
* implemented without the use of system calls at all.

The same API can exist on multiple systems and provide the same interface to applications while the implementation of the API itself can differ greatly from system to system.

The figure below shows relationship between a POSIX API, the C library, and system calls.

[![Figure 5.1 The relationship between applications, the C library, and the kernel with a call to printf().](figure_5.1_600.png)](figure_5.1.png "Figure 5.1 The relationship between applications, the C library, and the kernel with a call to printf().")

#### POSIX *

The most common APIs in the Unix world is based on POSIX. Technically, POSIX is composed of a series of standards from the [IEEE](https://en.wikipedia.org/wiki/Institute_of_Electrical_and_Electronics_Engineers) that aim to provide a portable operating system standard roughly based on Unix. Linux strives to be POSIX- and SUSv3-compliant where applicable.

On most Unix systems, the POSIX-defined API calls have a strong correlation to the system calls. Some systems that are rather un-Unix, such as Microsoft Windows, offer POSIX-compatible libraries. [p70]

#### The C Library *

The system call interface in Linux, as with most Unix systems, is provided in part by the C library.

The C library implements the main API on Unix systems, including:

* The standard C library
* The system call interface

The C library is used by all C programs and, because of C’s nature, is easily wrapped by other programming languages for use in their programs. The C library additionally provides the majority of the POSIX API.


From the application programmer’s point of view, system calls are irrelevant; all the programmer is concerned with is the API. Conversely, the kernel is concerned only with the system calls; what library calls and applications make use of the system calls is not of the kernel’s concern. Nonetheless, it is important for the kernel to keep track of the potential uses of a system call and keep the system call as general and flexible as possible.

A meme related to interfaces in Unix is "Provide mechanism, not policy". In other words, Unix system calls exist to provide a specific function in an abstract sense. The manner in which the function is used is not any of the kernel’s business.

### Syscalls

**System calls** (often called **syscalls** in Linux) are typically accessed via function calls defined in the C library.

* The functions can define zero, one, or more arguments (inputs) and might result in one or more side effects.
    *  Although nearly all system calls have a side effect (that is, they result in some change of the system’s state), a few syscalls, such as `getpid()`, merely return some data from the kernel.
* System calls also provide a return value of type `long` (for compatibility with 64-bit architectures) that signifies success or error.
    * Usually, although not always, a negative return value denotes an error.
    * A return value of zero is usually (not always) a sign of success.
    * The C library (when a system call returns an error) writes a special error code into the global `errno` variable, which can be translated into human-readable errors via library functions such as `perror()`.
* System calls have a defined behavior. (see the following example)

The system call `getpid()` is defined to return an integer that is the current process’s PID. The implementation of this syscall in the kernel is simple:

```c
SYSCALL_DEFINE0(getpid)
{
    return task_tgid_vnr(current); // returns current->tgid
}
```

The definition says nothing of the implementation. The kernel must provide the intended behavior of the system call but is free to do so with whatever implementation it wants as long as the result is correct. [p72]

`SYSCALL_DEFINE0` is simply a macro that defines a system call with no parameters (hence the 0). The expanded code looks like this:

<small>[include/linux/syscalls.h](https://github.com/shichao-an/linux/blob/v2.6.34/include/linux/syscalls.h##L285)</small>

```c
asmlinkage long sys_getpid(void);
```

* The `asmlinkage` modifier on the function definition is a directive to tell the compiler to look only on the stack for this function’s arguments. This is a required modifier for all system calls.
* The function returns a `long`. For compatibility between 32- and 64-bit systems, system calls defined to return an `int` in user-space return a `long` in the kernel.
* The naming convention taken with all system calls in Linux is: System call `bar()` is implemented in the kernel as function `sys_bar()`.

#### System Call Numbers

In Linux, each system call is assigned a unique **syscall number** that is used to reference a specific system call. When a user-space process executes a system call, the syscall number identifies which syscall was executed; the process does not refer to the syscall by name.

* When assigned, the syscall number cannot change; otherwise, compiled applications will break.
* If a system call is removed, its system call number cannot be recycled, or previously compiled code would aim to invoke one system call but would in reality invoke another.
* Linux provides a "not implemented" system call, `sys_ni_syscall()`, which does nothing except return `ENOSYS`, the error corresponding to an invalid system call. This function is used to "plug the hole" in the rare event that a syscall is removed or otherwise made unavailable.

The kernel keeps a list of all registered system calls in the system call table, stored in `sys_call_table`, on x86-64 it is defined in [arch/x86/kernel/syscall_64.c](https://github.com/shichao-an/linux/blob/v2.6.34/arch/x86/kernel/syscall_64.c).

The system call numbers are defined in the file [include/asm-generic/unistd.h](https://github.com/shichao-an/linux/blob/v2.6.34/include/asm-generic/unistd.h).

#### System Call Performance

System calls in Linux are faster than in many other operating systems, because of:

* Linux’s fast context switch times: entering and exiting the kernel is a streamlined and simple affair
* Simplicity of the system call handler and the individual system calls themselves

### System Call Handler

It is not possible for user-space applications to execute kernel code directly. They cannot simply make a function call to a method existing in kernel-space because the kernel exists in a protected memory space. Otherwise, system security and stability would be nonexistent.

User-space applications signal the kernel that they want to execute a system call and have the system switch to kernel mode, where the system call can be executed in kernel-space by the kernel on behalf of the application. This mechanism is software interrupt: incur an exception, and the system will switch to kernel mode and execute the **exception handler**. The exception handler in this case is actually the **system call handler**.

The defined software interrupt on x86 is interrupt number 128, which is incurred via the `int $0x80` instruction. It triggers a switch to kernel mode and the execution of exception vector 128, which is the system call handler. The system call handler is the aptly named function `system_call()`. It is architecture-dependent; on x86-64 it is implemented in assembly in `entry_64.S` ([arch/x86/kernel/entry_64.S](https://github.com/shichao-an/linux/blob/v2.6.34/arch/x86/kernel/entry_64.S)).

Recently, x86 processors added a feature known as *sysenter*, which provides a faster, more specialized way of trapping into a kernel to execute a system call than using the `int` interrupt instruction. Support for this feature was quickly added to the kernel. Regardless of how the system call handler is invoked, however, the important notion is that somehow user-space causes an exception or trap to enter the kernel.

#### Denoting the Correct System Call

For more details of Linux system call from the assembly perspective, see [Interfacing with Linux](../asm/index.md#interfacing-with-linux-system-calls).

Simply entering kernel-space alone is not sufficient: the system call number must be passed into the kernel.

On x86, the syscall number is passed to the kernel via the `eax` register:

* Before causing the trap into the kernel, user-space sticks in `eax` the number corresponding to the desired system call.
* The system call handler then reads the value from `eax`

The `system_call()` function checks the validity of the given system call number by comparing it to `NR_syscalls`. If it is larger than or equal to `NR_syscalls`, the function returns -`ENOSYS`. Otherwise, the specified system call is invoked:

<small>[arch/x86/kernel/entry_64.S#L487](https://github.com/shichao-an/linux/blob/v2.6.34/arch/x86/kernel/entry_64.S#L487)</small>

```gas
call *sys_call_table(,%rax,8)
```

Because each element in the system call table is 64 bits (8 bytes), the kernel multiplies the given system call number by eight to arrive at its location in the system call table. On x86-32, the code is similar, with the 8 replaced by 4.

[![Figure 5.2 Invoking the system call handler and executing a system call.](figure_5.2.png)](figure_5.2.png "Figure 5.2 Invoking the system call handler and executing a system call.")

#### Parameter Passing

In addition to the system call number, most syscalls require that one or more parameters be passed to them. User-space must relay the parameters to the kernel during the trap. The easiest way to do this is similar to how the syscall number is passed: The parameters are stored in registers. On x86-32, the registers `ebx`, `ecx`, `edx`, `esi`, and `edi` contain, in order, the first five arguments. In the unlikely case of six or more arguments, a single register is used to hold a pointer to user-space where all the parameters are stored.

The return value is sent to user-space also via register. On x86, it is written into the `eax` register.

### System Call Implementation

The actual implementation of a system call in Linux does not need to be concerned with the behavior of the system call handler. Thus, adding a new system call to Linux is relatively easy. The hard work lies in designing and implementing the system call; registering it with the kernel is simple.

#### Implementing System Calls

The first step in implementing a system call is defining its purpose and the syscall should have exactly one purpose. Multiplexing syscalls (a single system call that does wildly different things depending on a flag argument) is discouraged in Linux. `ioctl()` is an example of what *not* to do.

Think of the new system call’s arguments, return value, and error codes. The system call should have a clean and simple interface with the smallest number of arguments possible.The semantics and behavior of a system call are important; they must not change, because existing applications will come to rely on them. Many system calls provide a flag argument to address forward compatibility. The flag is not used to multiplex different behavior across a single system call (which is not acceptable), but to enable new functionality and options without breaking backward compatibility or needing to add a new system call. [p75]

Design the system call to be as general as possible with an eye toward the future. <u>The *purpose* of the system call will remain constant but its *uses* may change.</u> Remember the Unix motto: "Provide mechanism, not policy." [p75]

When you write a system call, you need to realize the need for portability and robustness, not just today but in the future.The basic Unix system calls have survived this test of time; most of them are just as useful and applicable today as they were 30 years ago!

#### Verifying the Parameters

System calls must carefully verify all their parameters to ensure that they are valid, legal and correct to guarantee the system’s security and stability. [p75]

One of the most important checks is the validity of any pointers that the user provides. Before following a pointer into user-space, the system must ensure that:

* The pointer points to a region of memory in user-space. Processes must not be able to trick the kernel into reading data in kernel-space on their behalf.
* The pointer points to a region of memory in the process’s address space.The process must not be able to trick the kernel into reading someone else’s data.
* The process must not be able to bypass memory access restrictions. If reading, the memory is marked readable. If writing, the memory is marked writable. If executing, the memory is marked executable.

The kernel provides two methods for performing the requisite checks and the desired copy to and from user-space. <u>Note kernel code must never blindly follow a pointer into user-space. One of these two methods must always be used.</u>

* `copy_to_user()` is for writing into user-space. It takes three parameters:
    * The first argument is the destination memory address in the process’s address space.
    * The second argument is the source pointer in kernel-space.
    * The third argument is the size in bytes of the data to copy.
* `copy_from_user()` is for reading from user-space. It is analogous to `copy_to_user()`. The function reads from the second parameter into the first parameter the number of bytes specified in the third parameter.

Both of these functions return the number of bytes they failed to copy on error. On success, they return zero. It is standard for the syscall to return `-EFAULT` in the case of such an error.

The following example `silly_copy()` uses both `copy_from_user()` and `copy_to_user()`. It copies data from its first parameter into its second. This is suboptimal in that it involves an intermediate and extraneous copy into kernel-space for no gain. But it helps illustrate the point:

```c
/*
* silly_copy - pointless syscall that copies the len bytes from
* ‘src’ to ‘dst’ using the kernel as an intermediary in the copy.
* Intended as an example of copying to and from the kernel.
*/
SYSCALL_DEFINE3(silly_copy,
                unsigned long *, src,
                unsigned long *, dst,
                unsigned long len)
{
    unsigned long buf;

    /* copy src, which is in the user’s address space, into buf */
    if (copy_from_user(&buf, src, len))
        return -EFAULT;

    /* copy buf into dst, which is in the user’s address space */
    if (copy_to_user(dst, &buf, len))
        return -EFAULT;

    /* return amount of data copied */
    return len;
}
```

Both `copy_to_user()` and `copy_from_user()` may block. This occurs, for example, if the page containing the user data is not in physical memory but is swapped to disk. In that case, the process sleeps until the page fault handler can bring the page from the swap file on disk into physical memory.

A final possible check is for valid permission. In older versions of Linux, it was standard for syscalls that require root privilege to use `suser()`. This function merely checked whether a user was root; this is now removed and a finer-grained “capabilities” system is in place.

The new system enables specific access checks on specific resources. A call to `capable()` with a valid capabilities flag returns nonzero if the caller holds the specified capability and zero otherwise. For example, `capable(CAP_SYS_NICE)` checks whether the caller has the ability to modify nice values of other processes. By default, the superuser possesses all capabilities and nonroot possesses none.

The following example is the `reboot()` system call. Note how its first step is ensuring that the calling process has the `CAP_SYS_REBOOT`. If that one conditional statement were removed, any process could reboot the system.

<small>[kernel/sys.c#L368](https://github.com/shichao-an/linux/blob/v2.6.34/kernel/sys.c#L368)</small>

```c
SYSCALL_DEFINE4(reboot, int, magic1, int, magic2, unsigned int, cmd,
		void __user *, arg)
{
	char buffer[256];
	int ret = 0;

	/* We only trust the superuser with rebooting the system. */
	if (!capable(CAP_SYS_BOOT))
		return -EPERM;

	/* For safety, we require "magic" arguments. */
	if (magic1 != LINUX_REBOOT_MAGIC1 ||
	    (magic2 != LINUX_REBOOT_MAGIC2 &&
	                magic2 != LINUX_REBOOT_MAGIC2A &&
			magic2 != LINUX_REBOOT_MAGIC2B &&
	                magic2 != LINUX_REBOOT_MAGIC2C))
		return -EINVAL;

	/* Instead of trying to make the power_off code look like
	 * halt when pm_power_off is not set do it the easy way.
	 */
	if ((cmd == LINUX_REBOOT_CMD_POWER_OFF) && !pm_power_off)
		cmd = LINUX_REBOOT_CMD_HALT;

	mutex_lock(&reboot_mutex);
	switch (cmd) {
	case LINUX_REBOOT_CMD_RESTART:
		kernel_restart(NULL);
		break;

	case LINUX_REBOOT_CMD_CAD_ON:
		C_A_D = 1;
		break;

	case LINUX_REBOOT_CMD_CAD_OFF:
		C_A_D = 0;
		break;

	case LINUX_REBOOT_CMD_HALT:
		kernel_halt();
		do_exit(0);
		panic("cannot halt");

	case LINUX_REBOOT_CMD_POWER_OFF:
		kernel_power_off();
		do_exit(0);
		break;

	case LINUX_REBOOT_CMD_RESTART2:
		if (strncpy_from_user(&buffer[0], arg, sizeof(buffer) - 1) < 0) {
			ret = -EFAULT;
			break;
		}
		buffer[sizeof(buffer) - 1] = '\0';

		kernel_restart(buffer);
		break;

#ifdef CONFIG_KEXEC
	case LINUX_REBOOT_CMD_KEXEC:
		ret = kernel_kexec();
		break;
#endif

#ifdef CONFIG_HIBERNATION
	case LINUX_REBOOT_CMD_SW_SUSPEND:
		ret = hibernate();
		break;
#endif

	default:
		ret = -EINVAL;
		break;
	}
	mutex_unlock(&reboot_mutex);
	return ret;
}
```

See `<linux/capability.h>` ([include/linux/capability.h](https://github.com/shichao-an/linux/blob/v2.6.34/include/linux/capability.h)) for a list of all capabilities and what rights they entail.

### System Call Context

The kernel is in process context during the execution of a system call ([Chapter 3](ch3.md#process-context)). The `current` pointer points to the current task, which is the process that issued the syscall.

In process context, the kernel is capable of sleeping (for example, if the system call blocks on a call or explicitly calls `schedule()`) and is fully preemptible. These two points are important:

1. **The capability to sleep** means that system calls can make use of the majority of the kernel’s functionality.
    * The capability to sleep greatly simplifies kernel programming ([Chapter 7](ch7.md))
    * Interrupt handlers cannot sleep and thus are much more limited in what they can do than system calls running in process context.
2. The fact that **process context is preemptible** implies that, like user-space, the current task may be preempted by another task.
    * <u>Because the new task may then execute the same system call, care must be exercised to ensure that system calls are reentrant.</u> This is the same concern that symmetrical multiprocessing introduces. (Synchronizing reentrancy is covered in [Chapter 9](ch9.md)).

<u>When the system call returns, control continues in `system_call()`, which ultimately switches to user-space and continues the execution of the user process.</u> ([Figure 5.2](figure_5.2.png))

#### Final Steps in Binding a System Call

It is trivial to register a (official) system call after it is written:

1. Add an entry to the end of the system call table. This needs to be done for each architecture that supports the system call. The position of the syscall in the table, starting at zero, is its system call number.
2. For each supported architecture, define the syscall number in `<asm/unistd.h>`.
3. Compile the syscall into the kernel image (as opposed to compiling as a module). This can be as simple as putting the system call in a relevant file in `kernel/`, such as `sys.c`, which is home to miscellaneous system calls.

For example of a fictional system call `foo()`. We want to add `sys_foo()` to the system call table. For most architectures, the table is located in `entry.S` and looks like this (The new system call is then appended to the tail of this list):

```c
ENTRY(sys_call_table)
.long sys_restart_syscall /* 0 */
.long sys_exit
.long sys_fork
.long sys_read
.long sys_write
.long sys_open /* 5 */
...
.long sys_eventfd2
.long sys_epoll_create1
.long sys_dup3 /* 330 */
.long sys_pipe2
.long sys_inotify_init1
.long sys_preadv
.long sys_pwritev
.long sys_rt_tgsigqueueinfo /* 335 */
.long sys_perf_event_open
.long sys_recvmmsg
.long sys_foo
```

Though not explicitly specified, the system call is then given the next subsequent syscall number (338, in this case).

* For each architecture you want to support, the system call must be added to the architecture’s system call table.
* The system call does not need to receive the same syscall number under each architecture, as the system call number is part of the architecture’s unique ABI.
* Usually, you would want to make the system call available to each architecture.
* Note the convention of placing the number in a comment every five entries; this makes it easy to find out which syscall is assigned which number.

Next, the system call number is added to `<asm/unistd.h>` like below:

```c
/*
* This file contains the system call numbers.
*/
#define __NR_restart_syscall 0
#define __NR_exit 1
#define __NR_fork 2
#define __NR_read 3
#define __NR_write 4
#define __NR_open 5
...
#define __NR_signalfd4 327
#define __NR_eventfd2 328
#define __NR_epoll_create1 329
#define __NR_dup3 330
#define __NR_pipe2 331
#define __NR_inotify_init1 332
#define __NR_preadv 333
#define __NR_pwritev 334
#define __NR_rt_tgsigqueueinfo 335
#define __NR_perf_event_open 336
#define __NR_recvmmsg 337
#define __NR_foo
```

Finally, the actual `foo()` system call is implemented. Because the system call must be compiled into the core kernel image in all configurations, in this example we define it in `kernel/sys.c`. You should put it wherever the function is most relevant; for example, if the function is related to scheduling, you could define it in `kernel/sched.c`.

```c
#include <asm/page.h>

/*
* sys_foo – everyone’s favorite system call.
*
* Returns the size of the per-process kernel stack.
*/
asmlinkage long sys_foo(void)
{
    return THREAD_SIZE;
}
```

Boot this kernel and user-space can invoke the `foo()` system call

#### Accessing the System Call from User-Space

The C library provides support for system calls. User applications can pull in function prototypes from the standard headers and link with the C library to use your system call. [p81]

Linux provides a set of macros for wrapping access to system calls. It sets up the register contents and issues the trap instructions. These macros are named `_syscalln()`, where *n* is between 0 and 6. The number corresponds to the number of parameters passed into the syscall, because the macro needs to know how many parameters to push into registers.

For example, consider the system call `open()`, defined as

```c
long open(const char *filename, int flags, int mode)
```

The syscall macro to use this system call without explicit library support would be:

```c
#define __NR_open 5
_syscall3(long, open, const char *, filename, int, flags, int, mode)
```

The application can simply call `open()`.

An macro has 2 + 2 × *n* parameters:

* The first parameter corresponds to the return type of the syscall.
* The second is the name of the system call.
* The remainder are type and name for each parameter in order of the system call.

The `__NR_open` ([arch/x86/include/asm/unistd_64.h#L19](https://github.com/shichao-an/linux/blob/v2.6.34/arch/x86/include/asm/unistd_64.h#L19)) is defined in `<asm/unistd.h>`; it is the system call number.

<u>The `_syscall3` macro expands into a C function with inline assembly;</u> the assembly performs the steps discussed in the previous section to push the system call number and parameters into the correct registers and issue the software interrupt to trap into the kernel. <u>Placing this macro in an application is all that is required to use the `open()` system call.</u>

For example of the `foo()` system call, we can use it from an application like this:

```c
#define __NR_foo 283
__syscall0(long, foo)

int main ()
{
    long stack_size;
    stack_size = foo ();
    printf ("The kernel stack size is %ld\n", stack_size);
    return 0;
}
```

#### Why Not to Implement a System Call

Adding new syscall is not encouraged, and you should otherwise exercise caution and restraint in adding one. [p82]o

The followings are pros and cons of implementing a new interface as a syscall:

Pros:

* System calls are simple to implement and easy to use.
* System call performance on Linux is fast.

Cons:

* You need a syscall number, which needs to be officially assigned to you.
* After the system call is in a stable series kernel, it is written in stone. The interface cannot change without breaking user-space applications.
* Each architecture needs to separately register the system call and support it.
* System calls are not easily used from scripts and cannot be accessed directly from the filesystem.
* Because you need an assigned syscall number, it is hard to maintain and use a system call outside of the master kernel tree.
* For simple exchanges of information, a system call is overkill.

The alternatives to implementing a syscall:

* Implement a device node and `read()` and `write()` to it. Use `ioctl()` to manipulate specific settings or retrieve specific information.
* Certain interfaces, such as semaphores, can be represented as file descriptors and manipulated as such.
* Add the information as a file to the appropriate location in sysfs

The slow rate of addition of new system calls is a sign that Linux is a relatively stable and feature-complete operating system. [p83]

### Conclusion

In this chapter discusses what system calls are and how they relate to library calls and the application programming interface (API). This includes how the Linux kernel implements system calls and the chain of events required to execute a system call: trapping into the kernel, transmitting the syscall number and any arguments, executing the correct system call function, and returning to user-space with the syscall’s return value. [p83]
