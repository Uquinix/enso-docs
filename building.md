# Building

> Run `git submodule update --init --recursive` to install the submodules before building

Enso can be built by using either LLVM or GNU toolchains.

For Arch Based Distributions:

`pacman -S make nasm`

For Debian Based Distributions:

`sudo apt-get install make nasm`

## With LLVM

Building Enso requires `lld` and `clang-13`.

For Arch Based Distributions

`pacman -S lld clang`

For Debian Based Distributions:

`sudo apt-get install lld clang`

Enso is build via `LLVM` by default, to build - run the following:

`make all`

> On some distros you might want to change the llvm version using `LLVM_VERSION=-13` or any other version.

## With GNU

In order to build Emsp with the GNU toolchain, you will need to build the x86_64-elf binutils. This can be done by running the script in `build/toolchain/gnu/build.sh`.

After building the binutils you can build Enso by running:

`make all TOOLCHAIN=gnu`

## Notes on macOS

If you use [Homebrew package manager](https://brew.sh), you can obtain the dependencies by running: 

`brew install nasm xorriso llvm binutils wget pkg-config sdl2`.

LLVM on Homebrew is keg by default, run `darwin.sh` to set these environment variables:

```sh
export PATH="/usr/local/opt/llvm/bin:$PATH"
export LDFLAGS="-L/usr/local/opt/llvm/lib"
export CPPFLAGS="-I/usr/local/opt/llvm/include"
```

It is recommended to set these in your shell-profile, as everytime your terminal window closes, these variables will be cleared.

Then build Enso by running:

`make all`

## Running Enso with Lotus

Lotus is based on the guts of QEMU, but with better debugging abilities, allowing for a faster and greater workflow.

To build Lotus, you must follow these simple steps.

```shell
mkdir -p build
cd build
../configure --target-list=x86_64-softmmu --enable-debug
make all -j$(nproc)
```

And to build Lotus (and automatically add to PATH)

```shell
sudo make install
```

After these steps, Lotus will just work. No setup required.