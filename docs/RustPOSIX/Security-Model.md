# Security Model

## Interface + Popular Paths

We define a narrow subset of Rust library calls that we deem necessary as our [interface](https://github.com/Lind-Project/safeposix-rust/wiki/Interface), which is further described in this wiki. We use this narrow subset so we an audit/fuzz the code that can reach the kernel, and determine that it adheres to the ["popular paths"](https://www.usenix.org/conference/atc17/technical-sessions/presentation/li-yiwen) philosophy.

## Memory Safety with Rust

The [Rust](https://www.rust-lang.org/) programming language is designed for safety while maintaining high-performance. It can also guarantee memory safety without garbage collection, which it achieves by using a borrow checker to validate references. This allows it to achieve the performance of languages without garbage collection, such as C/C++. Rust is especially valuable for safe concurrency.

Rust is designed to be memory safe, and it does not permit null pointers, dangling pointers, or data races in safe code. It instead uses an Option type to determine if a value is Some or None. Rust also introduces added syntax to manage lifetimes, and the compiler reasons about these through its borrow checker. 

[According to developers at Google](https://www.zdnet.com/article/chrome-70-of-all-security-bugs-are-memory-safety-issues/), 70% of software bugs are memory safety issues. We are using Rust for the development of SafePOSIX to control those issues.


## Additional Sandboxing

There are many features of a traditional Operating System that are unnecessary for our model. For example, our filesystem is confined to a single directory in the host system, and we should disallow access outside that directory.

Using a sandboxing program such as [Minijail](https://google.github.io/minijail/minijail0.1) helps narrow the set of allowed capabilities. In addition to sandboxing the file system, Minijail can disallow a set of system calls via seccomp filtering. This enables us to disallow obscure system calls that don't adhere to the popular paths at all, as well as system calls that are necessary for many exploits such as `fork` and `exec` that we don't need in our single process model.