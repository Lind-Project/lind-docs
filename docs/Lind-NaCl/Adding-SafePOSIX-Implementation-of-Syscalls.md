# Development Cycle:
Two main paths can be taken in implementing a new system call, defined in the Native Client section. In both these paths, a system call is defined in `Lind_GlibC`.

## GlibC (Lind_GlibC)

We customize the Standard C library used by the NaCl C compiler in GlibC to identify the system calls we implement in Lind and link any further references to our implementations.

First, a file in `Lind_GlibC/sysdeps/nacl/` named `X.c` should be created for a system call X. In this file, a function to implement the system call should be defined with the name `__X`. This definition should be followed by `weak_alias (__X, X)` to denote the reference. `libc_hidden_def(__X)` should also be used here if the header file for the function has `libc_hidden_proto(__X)`.

One can also choose to add both `libc_hidden_proto` and `libc_hidden_def` to the relevant files if `libc_hidden_proto` is not present in the header. For the functionality of these functions, one can look at the [`libc-symbols.h`](https://code.woboq.org/userspace/glibc/include/libc-symbols.h.html) file of the original Glibc source code. 

This file's full path (`sysdeps/nacl/X.c`) should then be added to the `override_list` in `make_sysd_rules.py` to make sure that this function is used.

This `__X` function should reference a function that can call the `NACL_SYSCALL` function. Such functions can be implemented in either `sysdeps/nacl/lind_syscalls.c` or `sysdeps/nacl/irt_syscalls.c` (after being defined in the respective header files).

In `irt_syscalls.c`, the convention is to implement this function as `nacl_irt_X` after defining the function as `*__nacl_irt_X` **in both the header and the .c file**. This is followed by an assignment: `__nacl_irt_X = nacl_irt_X` at the bottom of `irt_syscalls.c`. These function are normally used to call `NACL_SYSCALL` with a custom system call name.

The function should be defined `nacl_syscalls.h` after the proper assignment of the syscall number. 

The functions defined in `irt_syscalls.c` (`__nacl_irt_X`) should then be added to the `GLIBC_PRIVATE` part in `Lind_GlibC/elf/Versions` alongside other system calls to ensure their interception.

## Native Client (native_client)

We make changes in native_client to define the behavior of the system calls in Lind. The general principle is that we make the security checks related to the information regarding the state of the NaCl session before initiating system calls here. We then define the procedures for the calling of Posix implementations and thereby link our Restricted Python syscall implementations to Native Client.

The changes needed to be done in the Native Client is completely dependent on the preferred development path:

### 1) Using native_client as a relay:

In this path, the `NACL_SYSCALL(lind_api)` reaches the `NaClSysLindSyscall` function in `native_client/src/trusted/service_runtime/lind_syscalls.c` file directly. In this function; the parameters are parsed and are combined with the `cage_id`. Then, the `CallPythonFunc` function in `native_client/src/shared/platform/lind_platform.c` is called with the proper parameters. This function carries out the RPC, directly relaying the system call number coming from GlibC to the RePy alongside the proper arguments format.

No changes are necessary for any of the files in Native Client for this path.

### 2) Implementing parts of the system call in native_client:

First, the name of the implemented syscall in GlibC should be defined as the number assigned to the syscall in GlibC in the file `native_client/src/trusted/service_runtime/include/bits/nacl_syscalls.h` to allow for the use of the same name macro here.

Then, the `NACL_SYSCALL` being implemented should be added to the `SYSCALL_LIST` in `native_client/src/trusted/service_runtime/nacl_syscall_handlers_gen.py`, denoting the inputs used and linking the name on the GlibC side to the syscall name in NaCl.

Then, the referred NaCl function should be defined in `native_client/src/trusted/service_runtime/nacl_syscall_common.h` and implemented in `native_client/src/trusted/service_runtime/nacl_syscall_common.c`. In the implementation here, the necessary security check should be made, and parsing of information that must be one in C must be carried out. Then, a call must be made to one of the functions pointed by the `NaClDescVtbl` (more information available at `native_client/src/trusted/desc/nacl_desc_base.h `) if the syscall is related to one of the functions pointed there. Such a call goes through many loops but finally lands on the functions calling the Posix implementations. Then, the result should be parsed and returned in the proper format. Otherwise, a direct calling of "lind_" functions lind_platform.h/lind_platform.c should be carried out.
 
Lastly, the implementation of functions linking the NaCl to Posix should be made. For this, functions with the name `lind_X` (for a syscall named X) should be defined in `native_client/src/shared/platform/lind_platform.h` that include their standard parameters and the cage id. Also, macros for the name of the syscalls (`LIND_safe_fs_X` / `LIND_safe_net_X`) should be defined here to match the numbers assigned to the GlibC/NaCl macros.


## Rust + POSIX:

We implement syscalls here. For more details about SafePOSIX-Rust interface, please check the [SafePOSIX wiki](../RustPOSIX/Home.md).

This file is where RPC is accepted and syscalls are delivered to the dispatcher. Have the new syscall included here:
- `safeposix-rust/src/safeposix/dispatcher.rs`

The number assigned to the system call here must match the one assigned in GlibC.
Add the syscall's Lind implementation here:
- `safeposix-rust/src/safeposix/syscalls/fs_calls.rs` 
OR
- `safeposix-rust/src/safeposix/syscalls/net_calls.rs`
OR
- `safeposix-rust/src/safeposix/syscalls/sys_calls.rs`

## Important files (for a system call X):

### GlibC

- `Lind-GlibC/sysdeps/nacl/X.c`
- `Lind-GlibC/make_sysd_rules.py`
- `Lind-GlibC/elf/Versions`
- `Lind-GlibC/sysdeps/nacl/irt_syscalls.h`
- `Lind-GlibC/sysdeps/nacl/irt_syscalls.c`
- `Lind-GlibC/sysdeps/nacl/nacl_syscalls.h`

### Native Client

- `native_client/src/trusted/service_runtime/include/bits/nacl_syscalls.h`
- `native_client/src/trusted/service_runtime/nacl_syscall_handlers_gen.py`
- `native_client/src/trusted/service_runtime/nacl_syscall_common.c`
- `native_client/src/trusted/service_runtime/nacl_syscall_common.h`
- `native_client/src/shared/platform/lind_platform.c`
- `native_client/src/shared/platform/lind_platform.h`

For reference: 
- `native_client/src/trusted/desc/nacl_desc_base.h `

### Rust + POSIX

- `safeposix-rust/src/safeposix/dispatcher.rs`
- `safeposix-rust/src/safeposix/syscalls/fs_calls.rs`
- `safeposix-rust/src/safeposix/syscalls/net_calls.rs`