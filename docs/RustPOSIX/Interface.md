# SafePOSIX - Rust Interface

One element of our security model is to contain our kernel calls to the ["popular paths".](https://www.usenix.org/conference/atc17/technical-sessions/presentation/li-yiwen) To ensure this, SafePOSIX-Rust must only call to libraries (via the Rust standard libraries, and any crates) via paths that limit kernel calls to these popular paths.

The SafePOSIX-Rust interface exposes SafePOSIX to these libraries in five files:

1. file.rs - filesystem related calls
2. comm.rs - network related calls
3. timer.rs - time related calls
4. misc.rs - locks, serialization, etc.
5. pipe.rs - pipes based on Lock-Free Circular Buffer

These files make up our interface module, and are the only place libraries are imported via `use`. This allows us to fuzz this slimmer interface for kernel access to ensure the broader SafePOSIX code can only access the popular paths.

SafePOSIX is then built using this interface. No library imports via `use` are allowed in any of the files that implement SafePOSIX. Any attempts to import outside libraries will be rejected via code review.


## File

File.rs includes the following interface data structures and functions:

**OPEN_FILES** - a global locked hash-set of open files

**EmulatedFile** - a struct that includes: the filename (String), the filepath, the file object (Rust File w/ Lock), and the filesize (usize), it includes the following methods:
* new - Constructor
* close - Safely closes the emulated file
* readat - reads a file from offset location to a C-buffer
* writeat - writes from a C-buffer to a file at given offset
* readfile_to_new_string - reads entire file to new Rust String (used to restore metadata)
* writefile_from_string - writes to entire file from Rust String (used for metadata persistence)
* zerofill_at - writes a specified number of 0's to file
* as_fd_handle_raw_int - gets system fd number of open file

**openfile** - opens an EmulatedFile

**removefile** - safely removes a file

**listfiles** - returns a list of open files

It also publicly exposes the types PathBuf, Path, Component, OsString, and SyncLazy (used for globals) under our own aliases

## Comm

Comm.rs includes the following interface data structures and functions to work with Unix, IPv4, and IPv6 socket:

**GenSockaddr** - a generic socket address and can be one of three types: Unix, V4 (IPv4), or V6 (IPv6)
* port - access port
* set_port - modify the port
* addr - access the address
* set_addr - modify the address
* set_family - modify the family
* get_family - access the family
* path - access the path

**GenIpaddr** - a generic IP address and can be either V4 (IPv4) or V6 (IPv6)
* is_unspecified - check if an IP address is unspecified
* from_string - parse an IP address from a string

**SockaddrUnix** - a Unix socket address and contains the sun_family (family identifier) and sun_path (path) fields


**new_sockaddr_unix** - create a new Unix address
**gen_ud_path** - generate a unique Unix domain path

**V4Addr** - IPv4 address

**SockaddrV4** - IPv4 socket address, sin_family for family identifier, sin_port for port number, sin_addr for IPv4 address

**V6Addr** - IPv6 address, sin6_family for family identifier, sin6_port for port number, sin6_flowinfo for IPv6 flow information, sin6_addr for IPv6 address, sin6_scope_id for IPv6 scope identifier

**SockaddrV6** - IPv6 socket address

**Socket** - a socket and contains the refcnt (reference count) and raw_sys_fd (raw system file descriptor) fields
* new - create a new socket
* bind - bind socket to a specific address
* connect - connect to a remote address
* sendto - send data
* recvfrom - receive data
* recvfrom_nonblocking - receive data from a socket in non-blocking mode
* listen - listen for incoming connections
* set_blocking - set the socket to blocking mode
* set_nonblocking - set the socket to non-blocking mode
* accept - accept new connections
* nonblock_accept - accept a new non-blocking connection
* setsockopt - set options on a socket
* check_rawconnection - check the status of a raw socket connection
* drop -  release any system resources associated with the socket

## Misc

Misc.rs includes the following interface data structures and functions:

**log_to_stdout** - prints string to stdout

**log_to_stderr** - prints string to stderr

**log_from_ptr** - prints c-buffer to stdout

**fillrandom** - fills a buffer with random bytes from /dev/urandom

**fillzero** - fills a buffer with zeros

**new_hashmap** - creates a new hashmap

**charstar_to_ruststr** - converts a c-buffer to a Rust string


It also publicly exposes the types HashMap, RwLock, Arc under our own aliases.

### serde

misc.rs also exposes some of the serde crate for serialization. This includes:

**Serialize, Deserialize** - traits to be extended to structs for serialization

**serde_serialize_to_string** - function to serialize a data-structure to a Rust string

**serde_deserialize_from_string** - function to deserialize a Rust string back into the appropriate data structure.

## Timer

timer.rs includes the following interface data structures and functions:

**timestamp** - get system time as an unsigned 64-bit integer

**starttimer** - create a new timer

**readtimer** - read from created timer

**sleep_ms** - sleep for x milliseconds

It also publicly exposes the types Instant and Duration under our own aliases.

## Pipe

pipe.rs includes the following interface data structures and functions:

**new_pipe** -  creates a new instance of the EmulatedPipe struct with a specified size

**EmulatedPipe** - a struct that includes: the write end of a ring buffer, the read end of a ring buffer, the reference count for the write end, the reference count for the read end, size of ring buffer (usize), indicator whether the end of file has been reached.
* new_with_capacity - create a new EmulatedPipe instance with the specified size and initializes the other fields accordingly
* set_eof - set eof flag to true
* get_write_ref - return the current reference count for the write end
* get_read_ref - return the current reference count for the read end 
* incr_ref - increment the reference count for either the read or write end based on the provided flags 
* decr_ref - decrement the reference count for either the read or write end based on the provided flags 
* write_to_pipe - write length bytes from pointer into pipe
* read_from_pipe - read length bytes from the pipe into pointer

