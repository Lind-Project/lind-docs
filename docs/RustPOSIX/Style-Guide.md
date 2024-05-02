# General Development Guidelines

## Project Specific Guidelines

- All PRs should be merged to the Develop branch

- Any imports from the standard library or any crates should be done in an interface file


## Rust

Rust maintains a set of API Guidelines [here](https://rust-lang.github.io/api-guidelines/about.html). Before starting work on this project, it's important to review these guidelines.

For this project, we adhere to the official Rust style guide to ensure consistency and best practices in our code. For detailed information and guidelines, please refer to [the official Rust style guide](https://doc.rust-lang.org/stable/style-guide/index.html).

We particularly want to pay attention to documenting and handling anything labeled `unsafe`.


## SSL

SafePOSIX-Rust is a project under the maintenance of NYU Tandon's [Secure Systems Lab](https://ssl.engineering.nyu.edu/). The lab maintains [coding guidelines](https://github.com/secure-systems-lab/code-style-guidelines) for several languages. Though no official Rust guidelines currently exist, the guidlines for commenting as applied to Python also apply here. 

**General Recommendations**

* Write docstrings and comments before or as you write your code. You will never again understand your code better.
* Don't state the obvious! Don't spam! If someone else (including the future you) thinks it is less effort to decipher your code than to read its documentation, then you probably commented too much. Plus, more docs means more maintenance work.
* Code documentation that contradicts the code is worse than no code documentation. Always make a priority of keeping the docstrings and comments up-to-date when the code changes.


# Project Specific Style Guidelines

- No unnecessary whitespace.

- Files should end in newlines.

**No Imports (use statements)**

No modules from the stdlibrary or crates should be imported via `use`. Everything you need to implement SafePOSIX should be provided via the interface files. 

If you ever have to type the word `use` in non-interface files, **YOU ARE PROBABLY DOING SOMETHING WRONG.**


**If/Else Statements**

These generally should be on separate lines. There is some leeway if the else portion is very short and keeping it on the same line actually _improves_ readability.

**Locked Data Structures**

You will frequently need to use data structures such as the global FS_METADATA and CAGE_TABLE, as well as the file descriptor table within each cage.

It is worth assigning these on one line such as:

`let mut fdtable = self.filedescriptortable.write().unwrap();`

where you will use (_mut_/write) to change the DS and (~~mut~~/read) to simply access them.

**Return and Error Codes**

Syscall return codes should be clearly marked. For example, if a function returns 0, we should annotate it such as:

`0 //chdir has succeeded!`

Error codes should be set via the `syscall_error` function. For example:

`syscall_error(Errno::ENOTDIR, "chdir", "the last component in path is not a directory")`

**Currently Unsupported Operations**

If there is something that we are purposefully leaving blank but either is specified in POSIX or is something we will need to get to later (i.e. network interoperability inside an existing syscall), return a syscall_error with error code EOPNOTSUPP with an explanation string explaining what exactly is not implemented.