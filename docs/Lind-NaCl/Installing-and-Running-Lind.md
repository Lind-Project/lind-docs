# Installing and Running Lind

## Installation

Pull the image from DockerHub, if you haven't already
`docker pull securesystemslab/lind`

Creating a new container: 
`docker run --privileged --ipc=host --cap-add=SYS_PTRACE -it securesystemslab/lind /bin/bash`

## Compiling and Running a program with Lind
You cannot directly use `gcc` to compile programs for Lind. Instead, you can run the tool `lindsh` first, and use the commands `compile`, `gcc`, or `cc` in this special compiler.

#### 1. Compile a hello world! from source with lindsh
Compile the `hello.c` file under the `/home/lind/lind_project/tests/test_cases/` directory

```bash
$ lindsh
@[/home/lind/lind_project] lind> compile [absolutepath]/path/to/hello.c -o /home/lind/lind_project/hello.nexe 
```
#### 2. Copy the .nexe file to lindfs, source and destination arguments must be absolute paths.

If you’re inside lindsh: 
`cp /home/lind/lind_project/hello.nexe hello.nexe`

From container: 
`lindfs cp ~/the/path/ /home/lind/lind_project/hello.nexe hello.nexe`

The first argument is where your executable file is located in the container. The second argument is where you want to put your executable in the Lind file system. Note you have to give the executable file a name in the Lind file system. 
The `cp` command can also recursively copy directories--if your second argument is a directory, it will copy the directory and any of its contents recursively, their paths are still relative to the first argument.

#### 3. Check if the .nexe file has been copied over
After exiting `lindsh` you can use the `lindfs` tool to check if it exists in the lind filesystem. 
`lindfs find | grep 'hello'`

Alternatively, you can use `lindsh ls /[path]/` to run a ls command and locate the file.

If you see your executable file is under the path you designated before, it means that you have successfully copied the file into the Lind file system.

#### 4. Run the .nexe file
You can run the executable with `lind /[path]/hello.nexe` or `run /hello.nexe` if you are in `lindsh` (alternate aliases for this command are `r`, `exec`, and `execute`).
