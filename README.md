# llama.cpp Android Tutorial (GPU)

🎉 **2025 version of tutorial is available! (April 4, 2025)**

## Hardware Prerequisites

- Android phone powered by **Qualcomm SoCs: Snapdragon 8 Gen 1, 2, 3 and 8 Elite**
- Recommend by not require: memory > 12 GB

## Reference Links

- [Qualcomm Official Tutorial](https://www.qualcomm.com/developer/blog/2024/11/introducing-new-opn-cl-gpu-backend-llama-cpp-for-qualcomm-adreno-gpu)

- [llama.cpp](https://github.com/ggerganov/llama.cpp)

## Software Prerequisites

### Termux (on Android phone)

Download Termux through F-Droid:

- [Termux official website](https://termux.dev/en/index.html)
- [F-Droid](https://f-droid.org/en/)

Change repo for faster speed (optional):

```bash
termux-change-repo
```

Check [here](https://wiki.termux.com/wiki/Package_Management) for more help.

### Softwares in Termux

In Termux, run:

```bash
pkg install python cmake make ninja git wget
```

## Install NDK

Run these commands first:

```bash
cd ~ 
wget https://dl.google.com/android/repository/commandlinetools-linux-8512546_latest.zip && \ 
unzip commandlinetools-linux-8512546_latest.zip && \ 
mkdir -p ~/android-sdk/cmdline-tools && \ 
mv cmdline-tools latest && \ 
mv latest ~/android-sdk/cmdline-tools/ && \ 
rm -rf commandlinetools-linux-8512546_latest.zip 
```

After that, check whether your OpenJDK is available or not:

```bash
ls /data/data/com.termux/files/usr/lib/jvm/ # Default Path
```

If not, download OpenJDK (for here I download OpenJDK-17):

```bash
pkg install openjdk-17
```

Then, add `JAVA_HOME` environment variable

```bash
vim ~/.bashrc
```

```bash
export JAVA_HOME=/data/data/com.termux/files/usr/lib/jvm/java-17-openjdk # Or your own jdk path
```

Apply the changes:

```bash
source ~/.bashrc
```

Verify that `JAVA_HOME` is set correctly 

```bash
echo $JAVA_HOME
```

Finally, run this:

```bash
yes | ~/android-sdk/cmdline-tools/latest/bin/sdkmanager "ndk;26.3.11579264" 
```

## Install OpenCL Headers and ICD loaders

Clone OpenCL Headers:

```bash
mkdir -p ~/dev/llm 
cd ~/dev/llm 
 
git clone https://github.com/KhronosGroup/OpenCL-Headers && \ 
cd OpenCL-Headers && \ 
cp -r CL ~/android-sdk/ndk/26.3.11579264/toolchains/llvm/prebuilt/linux-x86_64/sysroot/usr/include 
```

Install the headers using CMake:

```bash
cmake -S OpenCL-Headers -B OpenCL-Headers/build -D CMAKE_INSTALL_PREFIX=OpenCL-Headers/install
```

```bash
cmake --build OpenCL-Headers/build --target install
```

Install OpenCL ICD Loader:

```bash
cd ~/dev/llm 
```

```bash
git clone https://github.com/KhronosGroup/OpenCL-ICD-Loader && \ 
cd OpenCL-ICD-Loader && \ 
mkdir build_ndk26 && cd build_ndk26 && \ 
cmake .. -G Ninja -DCMAKE_BUILD_TYPE=Release \ 
  -DCMAKE_TOOLCHAIN_FILE=$HOME/android-sdk/ndk/26.3.11579264/build/cmake/android.toolchain.cmake \ 
  -DOPENCL_ICD_LOADER_HEADERS_DIR=$HOME/android-sdk/ndk/26.3.11579264/toolchains/llvm/prebuilt/linux-x86_64/sysroot/usr/include \ 
  -DANDROID_ABI=arm64-v8a \ 
  -DANDROID_PLATFORM=24 \ 
  -DANDROID_STL=c++_shared && \ 
ninja && \ 
cp libOpenCL.so ~/android-sdk/ndk/26.3.11579264/toolchains/llvm/prebuilt/linux-x86_64/sysroot/usr/lib/aarch64-linux-android 
```

## Build llama.cpp with OpenCL backend

```bash
cd ~/dev/llm 
 
git clone https://github.com/ggerganov/llama.cpp && \ 
cd llama.cpp && \ 
mkdir build-android && cd build-android 
 
cmake .. -G Ninja \ 
  -DCMAKE_TOOLCHAIN_FILE=$HOME/android-sdk/ndk/26.3.11579264/build/cmake/android.toolchain.cmake \ 
  -DANDROID_ABI=arm64-v8a \ 
  -DANDROID_PLATFORM=android-28 \ 
  -DBUILD_SHARED_LIBS=OFF \ 
  -DGGML_OPENCL=ON 
 
ninja 
```

The executable files (such as `llama-cli` will be available in `build-android/bin`)
