# Architecture

![SafePOSIX Diagram](./assets/SafePOSIX%20Rust%20Diagram.jpg)

## Interface

Detailed [here](./Interface.md), this is the set of imported libraries that we are allowing for use while creating SafePOSIX.

## Dispatcher/RPC

The dispatcher receives system call requests from Native Client. Since SafePOSIX-Rust is loaded as a shared object, these RPC calls are direct function calls. 

The dispatcher function receives calls in the format: 

`dispatcher(cageid: i32, callnum: i32, arg1: Arg, arg2: Arg, arg3: Arg, arg4: Arg, arg5: Arg, arg6: Arg)`

Each call sends the cage ID number from which it is submitted and the syscall number as integers, followed by six arguments.

These six arguments are Unions, and have a set of allowed types (or None) which correspond to the types needed for the syscall. This allows us to use one dispatcher function in Rust for a variety of system calls, without having to serialize arguments.

The dispatcher then checks if the cage exists in the cage table, and if it doesn't, initializes a new cage. It then takes the cage object corresponding to that ID number, and calls the method corresponding to the sent call number. We use `match` to call the correct function based on call number.

The cage table is protected by a RW lock. All system calls access the cage table as read, except for fork, exec, and exit which will need to manipulate the table.

When the system call method returns, the dispatcher returns the return/error code to NaCl.

## Cage Objects

* Cage ID (integer)
* Current Working Directory (String)
* Parent ID (integer)
* File Descriptor Table (Locked Hashmap of integers -> Descriptor Enums)

### File Descriptor Table

The file descriptor table is a hash map of fd integers to our file descriptor representations. File Descriptors are implemented as an Enum that can correspond to five descriptor types (File, Stream, Socket, Pipe, Epoll). Each of these descriptor types is a struct with its corresponding fields which can be seen in [cage.rs](https://github.com/Lind-Project/safeposix-rust/blob/main/src/safeposix/cage.rs)

### System Calls

Each cage object has public methods corresponding to each system call. These calls are implemented either as filesystem related calls, system related calls, or network related calls in their respective files. 

Each system call either returns a return code, or an error code (which is generated from [the errno Enum](https://github.com/Lind-Project/safeposix-rust/blob/main/src/interface/errnos.rs)).

The following are a brief description of the public methods
- `get_next_fd` - get the next available fd number
- `load_lower_handle_stubs` - initialize standard in/out/err descriptors
- `insert_next_pipe` - given a pipe, insert it into the pipe table
- `add_to_fd_table`, `rm_from_fd_table`, and `changedir` are **UNUSED!**

## FS Metadata

Our metadata table (in the code, this is `FilesystemMetadata` struct) is similar to how the metadatatable from SafePOSIX-Repy operated.

The table is represented by a struct with fields:

* nextinode - usize
* dev_ud - u64
* inodetable - hashmap of usize to InodeEnum

The Inode Enum can describe a variety of Inode structs which include: File(generic), CharDev, Socket, Directory

This metadata is stored in the file `lind.metadata`, `init_fs_metadata()` will read and restore the metadata from this file if it exists, otherwise, use `blank_fs_init()`. 

# Major Changes from SafePOSIX-Repy

Moving to Rust has allowed us some helpful structural changes. These are some of the major changes from Repy to Rust:

### Locks

Instead of the lock object that is provided in Repy, we wrap our data structures in locks (as well as atomic reference counts). We've chosen to use RWLocks to give us the option of having multiple readers when allowed (there can only be one writer at a time.)

This allows us to unlock for the scope of the variable, instead of having to track lock objects and using them to bound sections of code.

### Globals/SyncLazy

Rust doesn't allow global data structures where initialization is required. However, it is useful for us to have these for things such as a the Cage Table and FSMetadata. We use [SyncLazy](https://doc.rust-lang.org/std/lazy/struct.SyncLazy.html), which is in the process of being added to the Rust Standard Library, to manage these data structures with global scope.

### Args via Unions
Since we will be loading SafePOSIX-Rust into Lind as a shared object, and not as script fed to an interpreter, we can change our RPC to direct functions calls. To do this, we still need to translate data types from C to data types from Rust. To do this we use [C-style Unions](https://doc.rust-lang.org/reference/type-layout.html#reprc-unions). Each argument is sent as an `Arg` union, which is interpreted in our dispatcher as the appropriate data type for its corresponding system call.

### Buffers/C-Structs 

Buffers/strings used to be packaged as Python strings and then sent to SafePOSIX-Repy. We found this to be extremely slow for large strings. We now are able to send the pointer to these buffers using the above `Arg` system. This is also true for buffers that need to be populated, and even Struct pointers.

For structs, we're able to declare C-style structs in Rust and populate them as we are populating the struct in C. This lets us avoid costly things like struct packing.

### Enums
We now represent objects like Inodes and File Descriptors as Rust Enums. This way we can use different structs for different object types while still describing the same base object. For example our file descriptors can represent either Files, Streams, Sockets, or Pipes, each with their own fields.

### Errnos

We use C-style enums to represent Errnos, which match an integer to a code-string. We've created a function to display proper error messages while returning Errnos for verbose logging.

### Serialization

We now use the crate `serde` for serialization instead of the original handspun serializer which was very slow.

