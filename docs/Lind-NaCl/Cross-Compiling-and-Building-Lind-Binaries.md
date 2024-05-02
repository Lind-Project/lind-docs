## Building binaries for Lind:
The scripts for cross-compilation and building of programs in `lind_project/tests/applications/` are in the executable `bootstrap_nacl` files in their respective folders. 

## Running Lind binaries:
After running the `bootstrap_nacl` file, the resulting binaries can be run like the .nexe files after being copied into the Lind file system.

## Information about specific programs:

### Python
The source code needed to build Python and the `bootstrap_nacl` file is located at `lind_project/tests/applications/python/`

The binary is built into the root folder of the source code (`lind_project/tests/applications/python/`).

The build process will give an error message at some point as it requires the running of some built intermediary binaries to generate certain required files. At that point, the `bootstrap_nacl` script will build a native version of Python into the `python-native` folder, copy the required files to the main folder and carry on the build process. 

In the end, there will be another error as the `make` process will try to run the cross-compiled code on the native machine. However, Python will already have been built at this point, and the resulting executable can be deemed valid.

The details about the building process can be found under [Issue 65](https://github.com/Lind-Project/lind_project/issues/65) in GitHub.

### Nginx
The source code needed to build Nginx and the `bootstrap_nacl` file is located at `lind_project/tests/applications/nginx/`

The executable binary is built into `objs` after the completion of `bootstrap_nacl`.

The details about the building process can be found under [Issue 63](https://github.com/Lind-Project/lind_project/issues/63) in GitHub.

### PostgreSQL
The source code needed to build PostgreSQL and the `bootstrap_nacl file` are located at `lind_project/tests/applications/postgres/`

The binary is built into the root folder of the source code (`lind_project/tests/applications/postgres/`).

The details about the building process can be found under [Issue 64](https://github.com/Lind-Project/lind_project/issues/64) in GitHub.

## Development Cycle:

The introduction of a new binary to Lind is a multi-step process. This process is largely dependent on a bootstrap file's debugging for each program, and therefore has unique steps for each application. Still, a good number of the unique problems faced are variations of a small subset of problems. The information below can serve as a guide in planning a roadmap for this process by detailing out the most common problems and their solutions.

### 0) Importing of the source code to the `lind_project`:

A folder with the application name including the source code needed to build the application should be put under `tests/applications`.

### 1) Initialization of General Variables in `bootstrap_nacl`:

This is the most fundamental part of the process as it is how we denote that the binary should be cross-compiled. Here, the relevant variables denoting compilers and the linker to be used should be updated to their `x86_64-nacl` counterparts with the given lines of code:

`export CC=x86_64-nacl-gcc`

`export CXX=x86_64-nacl-g++`

`export LD=x86_64-nacl-ld`

The flags for each of these should also be updated. The generic code used for this is:

`export CFLAGS='-g -fno-pie'`

`export CXXFLAGS='-g -fno-pie'`

`export LDFLAGS='-g -fno-pie'`

Please note that the flag values are flexible, and can be changed. Some programs will fail to build with certain flags, so this process should be undertaken alongside the debugging process. For example, the build process of `nginx` could only be completed after the removal of the `-fno-pie` flag from `CFLAGS`.

Lastly, the path variable should be set with:

`export PATH="/usr/bin:/usr/sbin:/bin:/sbin:$PATH"`

### 2) Configuration Process:

After the relevant variables are set, the configuration process should be handled with the given command in `bootstrap_nacl`:

`./<relative_path_of_the_configure_file>/configure <configuration_specific_vartiable_exports> <configuration_flags> --host=x86_64-nacl "$@"`

Here, the <configuration_specific_vartiable_exports> and <configuration_flags> parts are to be determined according to the functionality wanted and should be improved with the debugging of the `bootstrap_nacl` file.

<configuration_flags> are especially important for the errors denoting the unavailability of certain libraries in the lin system. These are frequent types of errors and can be solved by either denoting the omitting of the library with the `--without_library` flag or the introduction of the ai library to the lind system.

### 3) Make Process:

The `bootstrap_nacl` file should also include a `make` command at the end. As simple as the command is, it can sometimes require some filesystem operation beforehand (after configuration) in order to succeed (see step 4). With this, the bootstrap file should look like:

`<variable_declerations>`

`<configuration>`

`<filesystem_operations>`

`make`

### 4) Filesystem Operations:

Sometimes, the `make` process will depend on some files that are not present in the source code and can not be generated by make. This can be the root cause of numerous `make` errors especially if the configuration file had executed successfully. This is usually due to a bug in the source application surfaced in the cross_compilation process or corruption in the source code. 

The first step in approaching these problems should be the validation of the source code. Depending on the method used to complete Step 0, the gitignore commands may have caused some files to be left out. If this is the case, a git add with --force followed by a push can be sufficient to solve the problem.

If the source code is intact, the problem likely arises from a bug in the source application. These problems can sometimes be solved by creating empty dummy files with the required names, although this should not be assumed to be a general solution. At this stage, a careful program-specific debugging process is necessary.

### 5) Source Code Changes:

Sometimes, the `x86_64-nacl` compiler may not be able to complete the compilation processes, giving syntax errors. This usually requires small changes in the relevant source code files. Mostly, the required changes are mild. The problem can usually be solved with  `#ifndef`, `#define`, `#endif` blocks.

### Common Errors:

1. Some applications only accept specific cross-compilation hosts (in config flags) supported by the developers. In these cases, a good strategy is to select a UNIX-based system and denote it as the host, while keeping the `nacl` system variable declarations.

2. Some applications do not officially support cross-compilation, and therefore their configure files do not accept host system names. Omitting the flag should not cause a problem in these cases, but these applications can have other problems. Two of which are explained below:

3. Some applications will try to run their built libraries during the `make` process. This will fail as the binaries are built to be run on the lind system, not on the machine on which we build the application. The solution to this problem is application-specific. 

* Sometimes, the application does not actually need the running of these binaries for the build process. In these cases, this problem can be solved with specific configure flags restricting the running of built binaries if such an option is present.

* Sometimes, the binaries can be essential parts of this build process. A possible solution to this is to first build a native version of the application, use filesystem operations to copy these files to our main build path, and re-run the make command. This will allow for `make` to run natively compiled binaries. See the build process of Python (explained above) for an example.

4. Some applications will reference the values of the building system to learn information like the length of an `int`. In these cases, these values can be overridden to denote the correct value for Lind with a source code change.
