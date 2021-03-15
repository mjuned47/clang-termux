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

5) Download clang-tblgen and llvm-tblgen from clang-build folder
   Of this repository
   Then make it executable :
   chmod +x clang-tblgen llvm-tblgen

6) export PATH=$PATH:(Installed NDK Path)/bin

7) Now go to llvm-project and execute :
   cmake -S llvm -B build -G "Ninja" 
   -DLLVM_ENABLE_PROJECTS="clang" 
   -DCMAKE_INSTALL_PREFIX=(Installation Path) 
   -DCMAKE_BUILD_TYPE=Release 
   -DLLVM_TARGETS_TO_BUILD=AArch64 
   -DCMAKE_C_FLAGS="-fPIE -flto -Os" 
   -DCMAKE_CROSSCOMPILING=True 
   -DLLVM_TARGET_ARCH=AArch64 
   -DLLVM_ENABLE_PIC=False 
   -DLLVM_DEFAULT_TARGET_TRIPLE=aarch64-linux-android-gcc 
   -DCMAKE_C_COMPILER=(Installed NDK Path)/bin/aarch64-linux-android-gcc 
   -DCMAKE_CXX_COMPILER=(Installed NDK Path)/bin/aarch64-linux-android-g++ 
   -DLLVM_TABLEGEN=(Path)/llvm-tblgen 
   -DCLANG_TABLEGEN=(Path)/clang-tblgen 
   -DCMAKE_CXX_FLAGS_RELEASE="-Os -flto=full"

8) ninja -C build -j(no. of cores)
   For example: ... -j4
9) Compiling ...
10) after completion, install your compiled clang to prefix by :
    ninja -C build install
