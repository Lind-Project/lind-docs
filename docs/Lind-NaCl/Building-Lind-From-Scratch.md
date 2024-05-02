## Docker

Currently, it's much easier to run Lind using the pre-built docker images via dockerhub. Currently it is extremely difficult/not possible to build Lind natively, due to needing downgraded versions of `make` and `texinfo` to build Lind's version of glibc.

Alternatively, you can build a docker image yourself by recursively cloning the `lind_project` repository, and running `make latest`.

For detailed instructions for building on Docker: [install and build](./Installing-and-Running-Lind.md#installation)

## Build Script Dependencies

If you decide to instead build Lind on your host system, the following dependencies are required:
* gcc
* clang
* python2
* Rust (rustup, nightly)

Install rustup, and switch to `nightly`:
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup toolchain install nightly
rustup default nightly
```

If the nightly toolchain is already installed, you might face some errors on updating the compiler, like `could not rename component file`. 
Use the below to fix it.
```
rustup uninstall nightly
rustup install nightly
```

We are currently attempting to fix our build of `glibc` to use modern tooling, until then we need to use earlier versions of `make` and `texinfo`.

The following dependencies can be compiled from the source distributions
found in `docker/pkgs.tar.xz`

* make 3.82
* texinfo 4.13

## Usage

Clone the [main Lind repository](https://github.com/Lind-Project/lind_project) to `~/lind_project`, install the python dependencies using `pip2 install --user virtualenv`, and then run `make` from the home directory. This will give you various build options.
```bash
git clone https://github.com/Lind-Project/lind_project.git ~/lind_project
cd ~/lind_project
pip install -r requirements.txt
```
Under the hood, make calls `./src/mklind` to start the build script using the default environment.

```bash
./mklind [-ehv] [targets]
```

####  Options
    -e      Use custom environment instead of defaults for directory paths.
    -h      Display this help.
    -q      Silent build (only errors are printed).
    -v      Display script version.

#### Custom Environment
The default environment of `./mklind` consists of

```bash
export LIND_PREFIX="$HOME"
export LIND_BASE="$LIND_PREFIX/lind_project"
export LIND_SRC="$LIND_BASE/lind"
export NACL_SDK_ROOT="$REPY_PATH/sdk"
export PYTHON="python2"
export PNACLPYTHON="python2"
export LD_LIBRARY_PATH="/home/lind/lind_project/lind/lindenv/:/lib/glibc:"
```

as the default set of sane paths for the build.
You can also export these environment variables yourself and use `./mklind` with the `-e` flag to use custom paths for the build.
