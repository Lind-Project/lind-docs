# Lind Project Documentation

This repository contains the documentation for the Lind Project.  Lind lets
many different legacy programs run in the same address space in a much more
secure and efficient manner than exists with Linux, containers, OSVMs, etc.
today.  To do this Lind uses three main technologies to implement [cages](#lind-nacl), [a microvisor](#rustposix),and [3i](#3i).

Please navigate to the respective directories for more detailed information
about each project.

## Lind-NaCl 
(Current Cage Technology)

The documentation for `Lind with NaCl` can be found in the `docs/lind-nacl` directory. 

[View Lind Documentation](docs/Lind-NaCl/Home.md)

Lind introduces the  abstraction of a cage, which is an isolated execution 
component (analogous to a process) which can safely run within the same 
OS process / address space as other software.  A cage is a lightweight
abstraction which is both failure and memory isolated from other cages in
the same address space, while having a potentially uniquely tailored and
restricted interface to the operating system.  As such, each `cage’ is
provided similar memory and OS isolation as is obtained with a separate
OSVM communicating via RPC, but which can interact with other cages with an
overhead more similar to a function call.

### Table of Contents

* [Home](./docs/Lind-NaCl/Home.md)
* [Lind Architectural Description](./docs/Lind-NaCl/Lind-Architectural-Description.md)
* [Installing and Running Lind](./docs/Lind-NaCl/Installing-and-Running-Lind.md)
* [Building Lind From Scratch](./docs/Lind-NaCl/Building-Lind-From-Scratch.md)
* [Adding SafePOSIX Implementation of Syscalls](./docs/Lind-NaCl/Adding-SafePOSIX-Implementation-of-Syscalls.md)
* [Cross Compiling and Building Lind Binaries](./docs/Lind-NaCl/Cross-Compiling-and-Building-Lind-Binaries.md)
* [Debugging and Profiling Techniques](./docs/Lind-NaCl/Debugging-and-Profiling-Techniques.md)
* [Lind Tools (lind,-lindsh,-lindfs)](./docs/Lind-NaCl/Lind-Tools-(lind,-lindsh,-lindfs).md)
* [Lind with VSCode](./docs/Lind-NaCl/Lind-with-VSCode.md)
* [Lind's Test Suite](./docs/Lind-NaCl/Lind's-Test-Suite.md)
* [Running Applications](./docs/Lind-NaCl/Running-Applications.md)
* [Running the Lind LAMP Stack Demo](./docs/Lind-NaCl/Running-the-Lind-LAMP-Stack-Demo.md)

## RustPosix 
(Current Microvisor Implementation)

The documentation for the `RustPOSIX` can be found in the `docs/rustposix` directory.

[View RustPosix Documentation](docs/RustPOSIX/Home.md)

However, there must be a way to fork cages, read and write files,
communicate over the network, etc.  This is done by communicating with a
piece of software in the process called a microvisor.  A microvisor is
effectively similar to an operating system which lives in the same process
with all of the caged processes and which provides a POSIX interface.  Code
running within a cage believes it has its own virtual machine (much in the
same way a container does) because the abstraction it is provided appears
to be a full operating system.  A cage can fork and exec other cages,
manipulate files on disk or access the network, etc., all using standard
off-the-shelf legacy programs (except for recompilation for some cage
environments).

### Table of Contents

* [Home](./docs/RustPOSIX/Home.md)
* [Architecture](./docs/RustPOSIX/Architecture.md)
* [Interface](./docs/RustPOSIX/Interface.md)
* [Run Independently](./docs/RustPOSIX/Run-Independently.md)
* [Security Model](./docs/RustPOSIX/Security-Model.md)
* [Style Guide](./docs/RustPOSIX/Style-Guide.md)
* [Testing and Debugging](./docs/RustPOSIX/Testing-and-Debugging.md)

## 3i

More to come!

In order to make cages interact and work well together, another key concept
is that of the iPC (intra-process call) interposable interface (3i).  Using
3i provides a means for cages to securely and efficiently communicate with
each other.  Importantly, data is copied between cages in necessary cases,
providing meaningful isolation between cages, while having a speed similar
to a function call.  3i is used to provide POSIX interfaces between cages,
with a cage having the ability to interpose, modify, and mutate the
behavior of a 3i interface.  This enables fine grained security and access
control without breaking program behavior in unneeded cases.  The fast
switching time here provides the ability to realize a number of security,
utility and performance benefits which otherwise would be too expensive to
provide.  

