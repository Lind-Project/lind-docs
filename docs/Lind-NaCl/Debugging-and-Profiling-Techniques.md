## Debugging with GDB
We're able to debug any trusted code from Native Client using gdb. This can be done with the following steps:

1. Build Lind via `make`
1. Navigate to `lind/lindenv/fs/`
1. Run gdb on the sel_ldr in the bin directory i.e. `gdb ../bin/sel_ldr`
1. In GDB, set your breakpoints and run via `-r` with the arguments you would have supplied to Lind. i.e. `r -a -- "runnable-ld.so" --library-path "/lib/glibc" /hello.nexe`

To enable debug symbols when compiling RustPOSIX set this environment variable: `export DEBUG_RUSTPOSIX=true`

## Tracing Syscalls inside Lind

System Calls that are not simulated inside native_client can be traced by setting 
1. `export TRACING_ENABLED=1`
2. make -> rebuild native client
3. install -> install native client

Now the tracing is enabled and by default writes to strace_output.txt located at -> `/home/lind/lind_project/lind/lindenv/fs/`.
Tracing Falls back to `stderr` incase of issues with opening the destination file.
Output length of string arguments is limited to 30 characters by default and can be configured inside `native_client/src/trusted/service_runtime/nacl_syscall_strace.c`


## Tracing Untrusted Code Faults via the PC

When an error occurs in Untrusted code, Lind supplies a message such as: `** Signal 11 from untrusted code: pc=2fb4012970e3`

This program counter value can be used to trace the source of the error. This PC is made up of the Cage Location (first 4 digits) and the actual program counter (last 8 digits). The Cage Location can be discarded for the purpose of debugging.

You can use `addr2line` to pinpoint the location your code is failing. For example `addr2line -f -e executable_address 0xPC` can be run and will show where your error occurs.

However, if the code is failing in a loaded library (commonly glibc), you'll have to first find the address in the library. To do this, print out the log of the Lind run at high verbosity (`-v -v -v -v/ -vvvv`), and search for where the library was loaded. You should find an open call that links the library name to a file descriptor. Then, there should be a succeeding `mmap` call that loads that fd into a portion of memory.

You can find the library pc by subtracting the given program pc from the offset where your library was loaded into memory. Then run `addr2line` on the compiled library with that new PC.

## Linking GLIBC into GDB

Find your relative glibc load address as detailed in the previous section. The easiest way to do this is run the program with high verbosity and break on NaClSysOpen. You'll eventually see an open of libc such as `/lib/glibc/libc.so.990e7c45`. Note the FD it yields, and look down the log a bit to see a corresponding mmap using that FD number. Where this maps is the relative address. You should also note the offset it uses.

Find the text address of your glibc by using `readelf`. i.e. `readelf -S /lib/glibc/libc.so.990e7c45`. Note the text adddress with the offset you just recorded subtracted. If the text address is 0x0010700, and your offset is 0x0010000, use 0x700.

Now, combine the base address for the cage you're failing in, the relative glibc address, and the modified text address. Ie a base address of 0x4d2b00000000, a relative glibc address of 0x011f0000, and a text address of 0x700 would yield an address of 0x4d2b011f0700.

Next, in GDB, break before where your segmentation fault or the spot where you want to debug occurs. Input the command `add-symbol-file local-glibc-path calculated-load-address`. Note that this is the local address and not the SafePOSIX address that we noted in the earlier load.
You should now be able to put breakpoints in gdb for glibc.

## Debugging with RR

### Installation
Installation for Mozilla's `rr` is documented [here](https://github.com/mozilla/rr/wiki/Building-And-Installing). Below is a brief summary for setting up alongside Lind.

**Configure the Host**

On your host, outside of Docker, you need to turn on the perf event counter.
`$ sudo sysctl kernel.perf_event_paranoid=1`

You can apply the setup automatically on startup by running
```
$ sudo bash 
# echo 'kernel.perf_event_paranoid=1' > '/etc/sysctl.d/51-enable-perf-events.conf'
# exit
```
**Install Packages**

Inside Docker, add the following packages via pacman
`$ sudo pacman -S cmake capnproto community/python-pexpect`
You may need to update the database using: 
`$ sudo pacman -Syy`

**Pacman Errors**

You may need to update the archlinux keyring, to do so:
`$ sudo pacman -S archlinux-keyring`
`$ sudo rm -rf /etc/pacman.d/gnupg/*`
`$ sudo pacman-key --init`
`$ sudo pacman-key --populate archlinux`

**Installation**

1. clone from the git repo here: https://aur.archlinux.org/rr.git
2. cd into that directory
3. makepkg -sci
4. sudo pacman -U \*.pkg.tar.\*

**Using RR**

Like with GDB, we'll need to run this from the `lind/lindenv/fs/` folder. Run the following command to record with RR, where PATH/TO/BUILD is your build folder, and YOUR-NEXE-FILE is the compiled program you want to run. 
```bash
$ ~/PATH/TO/BUILD/bin/rr record ../bin/sel_ldr -a -v -v -v -v -- "runnable-ld.so" --library-path "/lib/glibc" /[yourfile.nexe]
```

Now you can replay and debug that recording with RR using: 
```bash
$ ~/PATH/TO/BUILD/bin/rr replay
```
Now in addition to GDB's base tools you have the ability to reverse (using GDB's syntax + r, so reverse continue would be `rc`). This is super handy if you want to set a watch point using `watch -l VAR_NAME` and reverse back to find where it was last changed.

## Cage IDs

Often seemingly inexplicable problems have been found out to be due to corrupted cage ID transmission. Make sure your cage IDs match from their NaCl RPC stubs in lind_platform.c with their interception in the SafePOSIX dispatcher (dispatcher.rs), either by enabling debugging in SafePOSIX or adding print statements.

## Common Errors

**NaClHostDescOpen: fstat failed?!?  errno 9
Using the wrong type of nexe (nacl-x86-32 on an x86-64 or vice versa)
or a corrupt nexe file may be responsible for this error.**

This one seems to come from a branch lagging behind, particularly Native Client. Make sure your branches are pulled and up to date.

## Profiling with perf and FlameGraph

Using `perf` for performance testing in Lind can be quite useful, especially when paired with [Brendan Gregg's FlameGraph implementation.](http://www.brendangregg.com/FlameGraphs/cpuflamegraphs.html#FlameGraph)
This can be cloned from github here: 
`git clone https://github.com/brendangregg/FlameGraph`

Instructions 
```
# cd FlameGraph
# perf record -F 99 -a --call-graph dwarf -- lind /your_progam_and_args.nexe args
# perf script | ./stackcollapse-perf.pl > out.perf-folded
# ./flamegraph.pl out.perf-folded > your_graph_name.svg
```

For Lind, it's important to use the `--call-graph dwarf` flag instead of just the `-g` flag because of it's utilization of the dwarf format.
