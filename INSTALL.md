## From binaries
No binary releases are provided at this time. Probably will be later.

## From source

### Non-Windows

```
mkdir build
cd build
cmake ..
make huestacean
```

### Windows

1. Open in Visual Studio using its cmake integration (File -> Open -> Cmake).
2. Build libhuestacean

# The following instructions are probably wrong

### Install gRPC
You need to have gRPC installed before you're able to build this. Building gRPC *could* be included as a build step in CMakeLists.txt, but the Windows Ninja requirement makes that a little annoying.

Instructions can be found here: https://github.com/grpc/grpc/blob/master/INSTALL.md

**NB:** There's a few problems following these instructions on Windows, at least with MSVC15, see below.

#### tl;dr

##### Non-Windows
```
 $ git clone -b 1.11.0 https://github.com/grpc/grpc
 $ cd grpc
 $ git submodule update --init
 $ make
 $ [sudo] make install
 ```

##### Windows
Install [Git](https://git-scm.com/) and [CMake](https://cmake.org/download/) if you haven't already.

Install [Chocolatey](https://chocolatey.org/), then use it to install the remaining prerequisites
```
choco install activeperl
choco install golang --version 1.9rc2
choco install yasm
choco install ninja
````

finally, clone and build grpc
```
> git clone --recursive -b v1.11.0 https://github.com/grpc/grpc
> cd grpc
> @rem Run from grpc directory after cloning the repo with --recursive or updating submodules.
> md .build
> cd .build
> @rem Only call the below if you're not already in the VS command prompt. Also mind it changes the working directory...
> call "C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Auxiliary\Build\vcvarsall.bat" x64
> @rem Specify cl.exe as the compiler or Ninja will use whatever it sees first, which may be a gcc if you have one in your PATH
> @rem Also, have to make sure -static is beign 
> cmake .. -GNinja -DCMAKE_BUILD_TYPE=Release -DCMAKE_C_COMPILER=cl.exe -DCMAKE_CXX_COMPILER=cl.exe
> cmake --build .
> ninja install
```

Some of the example targets in boringssl may not build. Doesn't matter.