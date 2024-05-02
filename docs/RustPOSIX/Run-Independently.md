Currently due to different computer architecture, we provide different approaches to make contributors can easily involve in.

# Devcontainer (Can be used in both MacBook and Linux)

Devcontainer, short for Development Container, is a feature provided by the Visual Studio Code Remote - Containers extension that allows you to develop code in an isolated Docker container environment. The purpose of this tool is to solve the 'it works on my machine' problem by standardizing the development environment to ensure consistency among development teams.

## General Steps

1. Install VS Code and Docker

2. Open VS Code and install `Remote - Containers`

3. Press `Ctrl+Shift+P` (Windows) / `command+Shift+P` (Mac) and type `Remote-Containers: Reopen in Container`

4. Then VS Code will reopen the project in the container.

# More general usage

## MacBook with M1/M2 chip Steps

1. Install Docker Desktop

2. Start Docker Desktop and open terminal

```
docker pull ubuntu:latest
docker run -it ubuntu:latest /bin/bash
```

3. Now we are in the docker container, install dependencies

```
apt update
apt install git
apt install curl
apt install gcc
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs/ | sh
rustup install nightly
. "$HOME/.cargo/env"
rustup default nightly
```

4. Clone git repo

```
cd home
git clone https://github.com/Lind-Project/safeposix-rust.git
cd safeposix-rust
git checkout mac-only
```

5. Build

```
cargo build
```

6. Run test suite

```
chmod +x gen_netdevs.sh 
./gen_netdevs.sh 
cargo test --lib
```

## Ubuntu

1. Setting environment 

```
apt update
apt install git
apt install curl
apt install gcc
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs/ | sh
rustup install nightly
. "$HOME/.cargo/env"
rustup default nightly
```

2. Clone git repo

```
cd home
git clone https://github.com/Lind-Project/safeposix-rust.git
```

3. Build

```
cargo build
```

4. Run test suite

```
chmod +x gen_netdevs.sh 
./gen_netdevs.sh 
cargo test --lib
```