
## Description

This wiki serves to be an architectural map of how Lind functions internally. This will serve as a handy guide for developers as the project continues.

## Overall Architecture

![Architecture](https://github.com/Lind-Project/lind_project/blob/ccf457da0427bcb623a36f981b774222872492b0/docs/lind-arch.png)

## Cages

A cage is a representation of a traditional process inside of Lind. It includes the 32-bit memory that is allocated to the cage, as well as data structures that assist in bookkeeping of the cage. These are implemented in both Native Client (NaClApp struct) and RustPOSIX (cage struct).


## Compilation

### NaCl-GCC + Toolchain

**Brief:**

NaCl-GCC, the Native Client compiler, produces binaries with software fault isolation preventing applications from performing system calls or executing arbitrary instructions. 

![Compilation Process](https://github.com/Lind-Project/lind_project/blob/ccf457da0427bcb623a36f981b774222872492b0/docs/rustposix-arch.jpg)

**System Calls:**

The Lind (modified NaCl) toolchain updates system calls to their Lind implementation for the compiled binaries. All Syscalls run through SafePOSIX and are modified to communicate via remote procedure call (RPC) to the RustPOSIX interface.

**glibc*: Lind's modified glibc, the GNU linker, handles system call replacement during compilation and generates an `nexe` file. The Internal Runtime (IRT) handler configures Lind and redirect immediately to Lind's RustPOSIX interface.

**Important Files:**
	
*   `lind_glibc/sysdeps/nacl/irt_syscalls.c` (IRT handler)

## Runtime

### Native Client

**Brief:**

The NaCl runtime environment manages the execution of the compiled `nexe` program. The nap struct is created by the `NaClAppCtor` function, and contains the application state. NaCl Runtime handles file descriptors as well as signals. Before the program is launched, caging setup is prepared in its own thread via `NaClCreateMainThread`/`NaClAppThreadSpawn`.


*   `native_client/src/trusted/service_runtime/nacl_syscall_common.c` (NaCl syscall interception)


**RPC:**

Lind packages system calls and sends them to the RustPOSIX kernel of Lind via the dispatcher.

**Important Files:**

* `native_client/src/shared/platform/lind_platform.c` (NaCl RPC)
* `native_client/src/trusted/service_runtime/`
    *   `sel_main.c` (start of Lind)
        - initializes various environment and runtime variables.    
    * `sel_ldr.c` (NaCl's `elf` file loader)    
    * `nacl_syscall_common.c` (main system call functions) 
    * `nacl_app_thread.c` (NaCl Runtime user thread state)

### RustPOSIX

**Brief:**

RustPOSIX, a re-design of the original [SafePOSIX](https://github.com/Lind-Project/nacl_repy) written in [Repy](https://github.com/SeattleTestbed/docs/blob/master/Programming/RepyV2Tutorial.md), handles system calls delivered via RPC from NaCl. Currently this handles filesystem, network, and system related calls. For detailed information, refer to the [RustPOSIX wiki page](../RustPOSIX/Home.md).

**System Calls:**

SafePOSIX system calls are implemented in RustPOSIX.

**Host Operating System:**

The safe system calls are executed via a limited API to request limited resources, such as CPU, memory, storage and network, from the host OS.

**Important Files:**

* `safeposix-rust/src/safeposix/`
    * `dispatcher.rs` (pairing)
        * routes the system call to the appropriate SafePOSIX implementation.
    * `syscalls/`
        * net
        * Sys
        * fs
* `safeposix-rust/src/interface/`
    * `file.rs` - filesystem related calls
    * `comm.rs` - network related calls
    * `timer.rs` - time related calls
    * `misc.rs` - locks, serialization, etc.


### Lind Filesystem
- We simulate a file system within Lind, making the Lind file system independent from the native file system. This means that native paths and LindFS paths are completely different and unrelated.
- The lind filesystem relates to the files that lind can access within the sandbox.
- The actual data for it is stored in the `lind_project/lind/lindenv/fs` directory
- Files inside the lind filesystem can also be be traversed, copied, updated, deleted using the `lindfs` command
- In the external fs, they are stored in non-encrypted form, thus file contents of files inside lindfs can also be directly accessed from outside. For example: `linddata.1961` represents the `test.txt` from inside the lind filesystem:

```bash
[1:0 0:430] 04:11:42 Thu Jan 25 [lind@26dfeb0d33b6: +1] ~/lind_project/lind/lindenv/fs                                                          
@(1:430) $ ls -l linddata.1961                                                                                                                  
-rw-r--r-- 1 lind lind 51 Jan 19 02:07 linddata.1961                                                                                            
3677/32022MB    11.58 4.70 2.45 6/1392 496      (0)                                                                                             
[1:0 0:431] 04:11:57 Thu Jan 25 [lind@26dfeb0d33b6: +1] ~/lind_project/lind/lindenv/fs                                                          
@(1:431) $ cat linddata.1961                                                                                                                    
This is a test of the wonderful fork call in lind.3572/32022MB  9.02 4.93 2.64 21/1420 522      (0)                                             
[1:0 0:432] 04:12:50 Thu Jan 25 [lind@26dfeb0d33b6: +1] ~/lind_project/lind/lindenv/fs                                                          
@(1:432) $ lindfs ls /test.txt                                                                                                                  
/test.txt                                                                                                                                       
1781/32022MB    7.68 5.18 2.83 3/1378 549       (0)                                                                                             
[1:0 0:432] 04:13:30 Thu Jan 25 [lind@26dfeb0d33b6: +1] ~/lind_project/lind/lindenv/fs                                                          
@(1:432) $ lind /bin/cat /test.txt                                                                                                
                                                                                                                                                
executing: [sel_ldr -a -- "runnable-ld.so" --library-path "/lib/glibc" /bin/cat /test.txt]                                        
[551,3629352960:04:13:39.654914] BYPASSING ALL ACL CHECKS                                                                                       
This is a test of the wonderful fork call in lind.1816/32022MB  6.81 5.08 2.82 5/1382 578       (0)
```

- Files in the external filesystem are labeled using `linddata+<inode of file inside lind fs>` For example, the inode of the `test.txt` file is `1961` inside the lind fs as can be seen below using the `lind /bin/ls -i <filename>` command -
```bash
[1:0 0:436] 04:15:48 Thu Jan 25 [lind@26dfeb0d33b6: +1] ~/lind_project/lind/lindenv/fs 
@(1:436) $ lind /bin/ls -i /test.txt

executing: [sel_ldr -a -- "runnable-ld.so" --library-path "/lib/glibc" /bin/ls -i /test.txt]
[754,2334926848:04:16:01.003393] BYPASSING ALL ACL CHECKS
1961 /test.txt
```

- More implementation details can be found in - `lind_project/src/safeposix-rust/src/safeposix/filesystem.rs`

### Other
- Lind Signal Handling Implementation documented in - `lind_project/docs/signal-handling-implementationl.md`