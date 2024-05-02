Lind comes with an automated test suite for testing whether C programs run the same natively as they do within Lind.

In short, the test suite does the following:
1. It compiles each specified file using both the standard gcc and the NaCl's gcc.
1. It copies the files compiled with NaCl's gcc into Lind's file system
1. For each specified file, the version compiled for Lind and the version compiled for native is run, and the outputs are compared. If the file is specified as nondeterministic, the outputs are sent to a python script which makes sure that the Lind compiled one is same compared to the native one. If any test fails, the exit code of the test suite will be nonzero.


# Using the Test Suite

Test suite is located under `~/lind_project/tests/test_cases/suite.sh` by default.
suite.sh should be invoked in the following way:

`./suite.sh [<non-deterministic files>] [-d <deterministic files>] [-v]`

Deterministic/Non-deterministic C files should be under a newline separated list. For example:

`./suite.sh nondet.txt [-d] dettests.txt [-v]`

Deterministic tests must be specified by `-d` flag. `-v` specifies verbosity.
For each nondeterministic test `example.c`, there should be a corresponding python file `example.py`, which will receive the output from the Lind compiled version and the output from the native compiled version (in that order) in argv, and exit with 0 on success and some other value on failure.