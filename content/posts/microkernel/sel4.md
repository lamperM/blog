---
title: "MicroKernel Learning: SeL4"
date: 2022-06-04T11:52:51+08:00
---

## seL4 Capabilities

In seL4, capabilities are stored in **C-space**. C-space is a hierarchical data structure very similar to **page table**.
* page table is a mapping from virtual address to physical address.
* C-space is a mapping from **object ID** to **capability**.
* Kernel object is made up of several **C-nodes**, just like a page table made up of individual page tables.
* Each C-nodes is an array of cap *slots*, which contain capability.

Inaccessible to userland, you can never hold an **actual capability**
* You can only hold a reference to capability, which pointers into C-space(slot addresses)
* These C-space addresses are called **CPTRs**

> You don't need to do the transform, because this is typically extracted in some libs.

Capabilities convey specific privilege (acces rights)
* Read, Write, Execute, GrantReply(`call`), Grant(`cap transfer`)

Main operations on capabilities:
* `Invoke`perform operation on object referred to by cap.
  * For example, map some frame into memory. You need to have capabilities to both the frame and address space.
* `Copy`|`Mint`|`Grant`: create copy of cap with **same/lesser** privilege.
* `Move`|`Mutate`: transfer to different address with **same/lesser** privilege.
  * Between C-space or within C-space.
* `Delete`: invalidate slot(cleans up object if this is the only cap to it)
* `Revoke`: delete any derived(eg. copied or minted) caps

### Capability Derivation
#### MINT OPERATION

The **Mint** operation creates a new, less powerful cap
* Can add badge
* Can strip access rights, eg `RW->RO`
```c
mint(dest, src, rights, badge)
```
* The first two arguement are **capability pointers(CPTR)** to a C-space(represented by C-node), which are references inside C-node. 
* The **destination C-node cap** must allow modification
* Then you have the rights and the *batch* of the new cap.

:pushpin: This is an alternative of sending addressed capabilities by **IPC operation**. 
That is what operating system do to set up protection domains for **user level process**.

#### COPY OPERATION

> Copy as a version of *Mint*.


&nbsp;
## seL4 Kernel Objects

In file `libsel4\include\sel4\objecttype.h`
```c
typedef enum api_object {
    seL4_UntypedObject,
    seL4_TCBObject,
    seL4_EndpointObject,
    seL4_NotificationObject,
    seL4_CapTableObject,
#ifdef CONFIG_KERNEL_MCS
    seL4_SchedContextObject,
    seL4_ReplyObject,
#endif
    seL4_NonArchObjectTypeCount,
} seL4_ObjectType;
```

&nbsp;
## seL4 System Calls

seL4 has `11` syscalls:

`Yield()`: invokes scheduler
* does NOT require a capability!

`Send()`,`Recv()` and variants/combinations thereof: IPC operations
* `Call()`,`ReplyRecv()`: usually invokes by client/server
* `Send()`, `NBSend()`: send-only and non-blocking version of it.
* `Recv()`, `NBRecv()`, `NBSendRecv()`
* `Wait()`, `NBWait()`, `NBSendWait()`

> We just use `Call()` normally, the others are only for bootstrapping protocols and exception handling.

Call() is atomic Send() + reply-object setup + Wait()
* cannot be simulated with one-way operations!

ReplyRecv() is NBSend() + Recv()

### Different object support different operations
#### ENDPOINTS

Endpoints support all 10 IPC variants.

#### NOTIFICATIONS

Notifications support:
* NBSend() - aliased as Signal()
* Wait()
* NBWait() - aliased as Poll()

#### OTHER OBJECTS
Other objects only supports `Call()` operation.
* Appear as (kernel-implemented) servers. If you invoking a method on an object, this is done by treating the object as a kernel-implemented server. And you invoke it with a `call()` operation just as you do a normal server invocation.
* Each of these kernel objects has a different kernel-defined protocol
  * operations encoded in message tag
  * parameters passed in message words
* Mostly hidden behind **syscall** wrappers, user do not need to know this details.


&nbsp;
## seL4 IPC

> IPC in seL4 is a way to realize **cross-domain** invocation.

seL4 IPC is not a mechanism for shipping data. Transfering data is axillary but not the primary purpose.

seL4 IPC is a protected procedure call, a user-controlled context switch(from clients context into server context).

&nbsp;
## seL4 Threads

### Creating a thread
1. Obtain a TCB object
2. Set attributes: V-space, C-space, fault endpoint, IPC buffer
3. Set Scheduling parameters: 
    * priority, scheduling context, timeout endpoint(maybe MCP)
4. Set architecture-related registers

### Threads and Stacks

Stacks are completely user-managed, kernel doesn't care!
> Kernel only preserves SP.. on context switch

Stack location, allocation, size must be managed by **userland**.

Kernel beware of stack overflow
