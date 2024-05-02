# lind, lindsh, and lindfs

## lind
`lind` or `../bin/sel_ldr -a -- "/lib/glibc/runnable-ld.so" --library-path "/lib/glibc"` runs an executable in Lind from `lindsh`. 
```
lind /hello.exe
```

#### Possible commands with lind
‘-E’ - add environment variable
```
lind -E [argument]
lind -E 
```

‘-v’ - increase the verbosity (can increase to up to -vvvv)
``` 
lind -v [executable file name]
lind -v /hello.nexe
```

‘r’ - run executable file
```
r [file name]
r /hello.nexe
```


## lindsh 
To build a program into executables in Lind, use NativeClient gcc (NaCl-gcc) to compile. Running `lindsh` will bring you into the lind file system and allow you to use lindprompt:
```
lindsh
>> compile [path to code] -o [output executable]
>> compile test/test_cases/hello.c -o hello.nexe 
```
To exit lindsh, run `exit`:
```
>> exit
```

You can also call cp (via lindfs) to copy programs in lindsh, or run (to run the Lind script):
```
cp /home/lind/lind_project/tests/test_cases/hello.nexe /hello.nexe
run /hello.nexe
```

## lindfs

To run commands in the lind file system, you run 
`lindfs [command] [arguments]` or `lind_fs_utils [command] [arguments]`

#### Possible commands with lindfs:

`cp` - copy files from host file system to lind filesystem
```
lindfs cp [local file on host file system] [path in lind file system]
lindfs cp /home/lind/lind_project/tests/test_cases/hello.nexe /home/lind/lind_project
```

`tree` - print lind file system file tree starting from specific directory
```
lindfs tree [directory path]
lindfs tree /lib
```

`ls` - list of contents in lind file system directory
```
lindfs ls [directory path]
lindfs ls /
```

`mkdir` - create a lind file system directory in current location
```
lindfs mkdir [directory name]
lindfs mkdir jkoe
```

`update` - copy host file system files in a given directory into lind file system directory (except for identical files)
```
lindfs update [directory path on host file system] [directory path to copy files to on lind file system]
lindfs update /home/lind/lind_project/tests/test_cases /home/lind/lind_project/jkoe
```

`rm` - delete a file in lind file system
```
lindfs rm [file name]
lindfs rm 
```

`rmdir` - delete a directory in lind file system
```
lindfs rmdir [directory name]
lindfs rmdir /home/lind/lind_project/jkoe
```

`deltree` - delete entire directory including its contents in lind file system
```
lindfs deltree [name of lind directory]
lindfs deltree /home/lind/lind_project/jkoe
```

`format` - make a new blank file system, **removing the current one**
```
lindfs format 
```

