# clang-termux
Clang-13 for AARCH64/ARM64

# How to build(cross-compile) clang for AArch64 on x86_64 Host ?

1) clone llvm-project :
   git clone https://github.com/llvm/llvm-project --depth 1

2) Download Android NDK : 
   https://developer.android.com/ndk/downloads
   I used r22 stable version(android-ndk-r22-linux-x86_64.zip)

3) unzip NDK and build clang :
   Go to NDK folder(android-ndk-r22) and execute 
   build/tools/make-standalone-toolchain.sh --arch=arm64 --toolchain=aarch64-linux-android
   --install-dir={location where you want to install ndk}

4) Install dependency :
   sudo apt install cmake ninja-build -y
   (Install other dependency if needed)

5) Download clang-tblgen-13 and llvm-tblgen-13 from clang-build folder
   Of this repository
   Then make it executable :
   chmod +x clang-tblgen-13 llvm-tblgen-13

6) export PATH=$PATH:(Installed NDK Path)/bin

7) Now go to llvm-project and execute :
   cmake -S llvm -B build -G "Ninja" 
   -DLLVM_ENABLE_PROJECTS="clang;compiler-rt;lld" 
   -DCMAKE_INSTALL_PREFIX=(Installation Path) 
   -DCMAKE_BUILD_TYPE=Release 
   -DLLVM_TARGETS_TO_BUILD=AArch64 
   -DCMAKE_C_FLAGS="-fPIE -Os" 
   -DCMAKE_CROSSCOMPILING=True 
   -DLLVM_TARGET_ARCH=AArch64 
   -DLLVM_ENABLE_PIC=False 
   -DLLVM_DEFAULT_TARGET_TRIPLE=aarch64-linux-android-gcc 
   -DCMAKE_C_COMPILER=(Installed NDK Path)/bin/aarch64-linux-android-gcc 
   -DCMAKE_CXX_COMPILER=(Installed NDK Path)/bin/aarch64-linux-android-g++ 
   -DLLVM_TABLEGEN=(Path)/llvm-tblgen-13 
   -DCLANG_TABLEGEN=(Path)/clang-tblgen-13 
   -DCMAKE_CXX_FLAGS_RELEASE="-Os"

8) ninja -C build -j(no. of cores)
   For example: ... -j4
9) Compiling ...
10) after completion, install your compiled clang to prefix by :
    ninja -C build install

# Issues :
 1) You might get weird error , it may be because of old tblgen files that we specify in cmake.

 Solution :
   To get llvm-tblgen and clang-tblgen files :
   - Remove c_compiler and cxx_compiler from above cmake command which was targeting ndk compiler
   - dont specify tblgen files in above cmake command. These two files will be created during compilation
   - export CC=gcc and export CXX=g++ (make sure you have gcc installed), you can use clang instead
   - final cmake command will look like this :
       cmake -S llvm -B build -G "Ninja" 
         -DLLVM_ENABLE_PROJECTS="clang;compiler-rt;lld" 
         -DCMAKE_INSTALL_PREFIX=(Installation Path) 
         -DCMAKE_BUILD_TYPE=Release 
         -DLLVM_TARGETS_TO_BUILD=AArch64 
         -DCMAKE_C_FLAGS="-fPIE -Os" 
         -DCMAKE_CROSSCOMPILING=True 
         -DLLVM_TARGET_ARCH=AArch64 
         -DLLVM_ENABLE_PIC=False 
         -DCMAKE_C_COMPILER=gcc 
         -DCMAKE_CXX_COMPILER=g++
         -DCMAKE_CXX_FLAGS_RELEASE="-Os"
    -start build ( ninja -C build)
    -total 3200 files will compile, once you have 2000 files compiled, stop compilation(ctrl+c)
     and check llvm-tblgen and clang-tblgen inside build/bin folder
    - llvm-tblgen will be produced when 500-700 files compiled (approx.)
    - clang-tblgen will be produced when 1800-2000 files compiled (approx.)
2) we can only cross compile to stage-1, because in stage-2 compilation
   compiled clang at stage 1 will be used for stage-2 compilation. 
   And AArch64 binary cannot be run natively on x86 host.
   
   Do you think qemu will help ?
   
   No. Binary compiled by ndk requires /system/bin/linker64 file which is only available in mobile
   So even if you use/install qemu for stage-2 compilation,
   You will get error(/system/bin/linker64 : no such file)

     
