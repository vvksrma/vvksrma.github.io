---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "Linux scheduler and context switch"
subtitle: ""
summary: "In this blog, I will be discussing how linux performs scheduling for the x86_64 architecture. I have tried to trace the function call path for context switches."
authors: []
tags: [Linux, Kernel, Scheduler, Context Switch, x86_64]
categories: []
date: 2024-05-05T21:12:47+05:30
lastmod: 2024-05-05T21:12:47+05:30
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---


In this blog, I will be discussing how linux performs scheduling for the x86_64 architecture. I have tried to trace the
function call path for context switches.


## How does the scheduler come into play?

The kernel provides the following ways to enter the scheduler function. These are mentioned within the kernel code as a comment to the __schedule function.
```
 * __schedule() is the main scheduler function.
 *
 * The main means of driving the scheduler and thus entering this function are:
 *
 *   1. Explicit blocking: mutex, semaphore, waitqueue, etc.
 *
 *   2. TIF_NEED_RESCHED flag is checked on interrupt and userspace return
 *      paths. For example, see arch/x86/entry_64.S.
 *
 *      To drive preemption between tasks, the scheduler sets the flag in timer
 *      interrupt handler scheduler_tick().
 *
 *   3. Wakeups don't really cause entry into schedule(). They add a
 *      task to the run-queue and that's it.
 *
 *      Now, if the new task added to the run-queue preempts the current
 *      task, then the wakeup sets TIF_NEED_RESCHED and schedule() gets
 *      called on the nearest possible occasion:
 *
 *       - If the kernel is preemptible (CONFIG_PREEMPTION=y):
 *
 *         - in syscall or exception context, at the next outmost
 *           preempt_enable(). (this might be as soon as the wake_up()'s
 *           spin_unlock()!)
 *
 *         - in IRQ context, return from interrupt-handler to
 *           preemptible context
 *
 *       - If the kernel is not preemptible (CONFIG_PREEMPTION is not set)
 *         then at the next:
 *
 *          - cond_resched() call
 *          - explicit schedule() call
 *          - return from syscall or exception to user-space
 *          - return from interrupt-handler to user-space
 *
 * WARNING: must be called with preemption disabled!
```

## How does context switch actually take place?
The following two points need to be noted:

- In x86, function calls are implemented using stack frames where caller and callee function perform specific duties of saving the register states(execution state).
- When the scheduler is called from the kernel mode execution, the function call history exists on the kernel stack frame.

The function which is first called to perform the context switch is the **__schedule** function.
It performs the following functions:

- #### pick_next_task:
    - The next task is picked using the "pick_next_task" function which takes the ready queue as the parameter and returns the task_frame of the next task to be scheduled.

- #### context_switch:
    - The "context_switch" function takes the prev and next task_structs.
    - It checks wether we need to switch the mm. Depending upon the nature of the outgoing and incoming contexts, we may need to do some handling of the vma's. The kernel follows the following rules:
    ```
    /*
         * kernel -> kernel   lazy + transfer active
         *   user -> kernel   lazy + mmgrab() active
         *
         * kernel ->   user   switch + mmdrop() active
         *   user ->   user   switch
    */

    ```
    -    In the case of kernel->kernel, we do not need to change the vma's as the kernel vmas are shared across contexts. In the case of user->kernel, we can just use the vma's of the outgoing context since we know that kernel vma's are shared.

- #### switch_to:
    - The kernel takes a lock on the run queue before calling this function. This prevents some wakeup task adding to the runqueue while the context switch is going on.
    - This takes the parameters(prev, next, prev), where prev is current context, next is the context to be scheduled.
    - Let us for the sake of understanding assume that our current process(prev) is P, incoming context(next) is Q. Also let us assume that Q was descheduled earlier calling R. 
    - **We need to understand that the when this(switch_to) function returns, we have completed the context switching and are in context of Q. Thus on returning our current context is the context pointed by "next" before the function call**
    - **We also need to allot the value of "prev" correctly now to context that was just descsheduled( P ). This is important because the earlier value of prev when Q was deschuled was Q itself!. And according to x86 function calling  conventions, we would have restored all the previous values i.e previous=Q, next=R**
    - Thus switch_to takes 3 parameters, and uses one "prev" for calling __switch_to_asm(next,prev), and the other "prev" for storing the returned value(which will be P).
    
- #### __switch_to_asm:
    - This function is inside the file: "arch/x86/entry/entry_64.S"
    - This is an assembly function. So accordingly we perform the callee duties of saving the following registers, inside the stack of P:
    ```
        pushq   %rbp
        pushq   %rbx
        pushq   %r12
        pushq   %r13
        pushq   %r14
        pushq   %r15
    ```
    - Following this, we switch the stack pointers. The stack pointer for the next(Q) is restored to this exact same point(where P is), however inside the Q stack.
    ```
        movq    %rsp, TASK_threadsp(%rdi)
        movq    TASK_threadsp(%rsi), %rsp
    ```
    - We then restore the registers "Inside the context of Q, from the stack of Q". These must have been stored when Q was switching to R.
    ```
        popq    %r15
        popq    %r14
        popq    %r13
        popq    %r12
        popq    %rbx
        popq    %rbp
    ```
    - We must note that the top of our stack will now hold the old rbp(for the call of __switch_to_asm from __switch_to) and the return address to go to the instruction after __switch_to_asm was called.
    - We then use a jmp instruction to execute __switch_to. This function performs the bulk of the switching work.

- #### __switch_to:
    - This function performs the bulk of the switching work.
    - We the return from this function directly to the instruction after call of __switch_to_asm. This is because the return address is configured to do so as mentioned in the previous instruction.


## Handling newly created processes.
- Previously we discussed the case where the scheduler switches from P to Q, where Q had previously been scheduled out calling some other context R.
- However what happens if we schedule a newly created context(Done by forking)?! We need some special handling for such cases.
- This is taken care of during the fork system call. The following snippet inside *arch/x86/kernel/process.c* does the following task:
```
    childregs = task_pt_regs(p);
    fork_frame = container_of(childregs, struct fork_frame, regs);
    frame = &fork_frame->frame;

    frame->bp = encode_frame_pointer(childregs);
    frame->ret_addr = (unsigned long) ret_from_fork;

```
- The fork system calls artificially puts the scheduler tail code onto the stack frame. Thus when the context switches into this, it will trap onto this code. "ret_from_fork" is the code that is used
```
call    schedule_tail 
```

## Conclusion

The linux scheduler is very complex entity that does a lot apart from simple context switches.
Important Kernel structures like rcu locks also rely on the scheduler to perform certain tasks.
It is responsible for a lot of efficiency in the kernel and is a very important subsystem. I have tried
to simplify the context switch process in this blog.