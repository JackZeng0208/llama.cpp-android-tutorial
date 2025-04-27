# llama.cpp Android Tutorial (Adreno OpenCL Backend)

## Note Updates

- 09/08/2023: First version of llama.cpp android tutorial is available

- **(NEW!)** 04/27/2025: 2025 version of tutorial is available
  - Also including `llama-cpp-python` + custom-built `llama.cpp` tutorial

## Highlights

- Deploying `llama.cpp` on an Android device and running it using the Adreno GPU.
- Utilizing `llama-cpp-python` with a custom-built `llama.cpp` version that supports Adreno GPU with OpenCL:
  - Enables large-scale inference evaluation directly on Android.
  - Provides a solid foundation for developing your own Android LLM applications.

## Hardware Prerequisites

- An Android device with a **Qualcomm Snapdragon SoC** (Snapdragon 8 Gen 1, Gen 2, Gen 3, or 8 Elite)
  - Recommended (but not required): more than 12 GB of RAM
- Optional: an Ubuntu or Mac computer for building

## Hardware Information

This tutorial is based on:

- Android Phone: OnePlus 13
  - SoC: Qualcomm Snapdragon 8 Elite
  - RAM: 16 GB
- Ubuntu Laptop: Kubuntu 24.04
  - Note #1: this guide is based on Ubuntu, but the steps are similar on MacOS and have been tested on a MacBook Pro.
  - Note #2: If you don't have a Linux laptop, then virtual machine also works

## Reference Links

- [Qualcomm Official Tutorial](https://www.qualcomm.com/developer/blog/2024/11/introducing-new-opn-cl-gpu-backend-llama-cpp-for-qualcomm-adreno-gpu)

- [llama.cpp](https://github.com/ggerganov/llama.cpp)

- [llama.cpp OpenCL Backend](https://github.com/ggml-org/llama.cpp/blob/master/docs/backend/OPENCL.md)

- [llama-cpp-python: how to use custom-built llama.cpp](https://github.com/abetlen/llama-cpp-python/issues/1070)

- [Build llama.cpp locally](https://github.com/ggml-org/llama.cpp/blob/master/docs/build.md)

## Software Requirements

### Termux (Android Device)

Install Termux via F-Droid:

- [Official Termux Website](https://termux.dev/en/index.html)
- [F-Droid](https://f-droid.org/en/)

(Optional) To improve download speeds, you can change the repository:

```bash
termux-change-repo
```

**Important:** Avoid installing Termux from the Google Play Store.

### Ubuntu Laptop

Required software:

- `python`, `cmake`, `make`, and `ninja`
  - Recommended: Install Python via `miniconda`
  - Guide: [Installing Miniconda](https://www.anaconda.com/docs/getting-started/miniconda/install)
- A C/C++ compiler (e.g., `clang`)
- Java Development Kit (in this case, `openjdk-21-jdk`)

Install them using the following command:

```bash
sudo apt install cmake make ninja-build clang openjdk-21-jdk
```

## Enable Termux Support for OpenCL + Adreno GPU

First, update termux packages:

```bash
pkg update && pkg upgrade
```

Then, download required libraries:

```bash
pkg install clinfo ocl-icd opencl-headers
```



### Option 1: For most of the Android Phone

Add dynamic link library `libOpenCL.so` path into `LD_LIBRARY_PATH` variable

```bash
vim .bashrc
```

```bash
# In .bashrc file, add:
export LD_LIBRARY_PATH=/vendor/lib64:$PREFIX/lib
```

For some older phone, the location of `libOpenCL.so` might be:

```bash
export LD_LIBRARY_PATH=/system/vendor/lib64:$PREFIX/lib
```

After that:

```bash
source ~/.bashrc
```

Test if it works:

```bash
clinfo
```

If it returns:

```
Number of platforms                               1
  Platform Name                                   QUALCOMM Snapdragon(TM)
...
```

It means OpenCL driver is loaded **successfully** 

If it returns:

```bash
Number of platforms                               0
...
```

It means OpenCL driver is **not detected**, and you may follow Option 2 below

### Option 2: For OnePlus 13 (or other phone)

Copy both `libOpenCL.so` and `libOpenCL_adreno.so` into your termux app

```bash
# The destination can be any other directory in your termux
cp /vendor/lib64/{libOpenCL.so, libOpenCL_andreno.so} ~/
```

Add the directory path containing these two libraries to the `LD_LIBRARY_PATH` environment variable

```bash
vim ~/.bashrc
```

```bash
# In my case, $HOME is the location of your directory containing two .so files
# In .bashrc file, add:
export LD_LIBRARY_PATH=$HOME:$LD_LIBRARY_PATH
```

```bash
source ~/.bashrc
```

Now, test it again:

```bash
clinfo
```

If it returns:

```
Number of platforms                               1
  Platform Name                                   QUALCOMM Snapdragon(TM)
...
```

It means OpenCL driver is loaded **successfully** 

## Building llama.cpp with OpenCL Backend

There are two options available:

1. Option 1: Build on Laptop and send it to Android phone
2. Option 2: Build on Android phone directly
   1. As of April 27, 2025, `llama-cpp-python` does not natively support building `llama.cpp` with OpenCL for Android platforms. This means you'll have to compile `llama.cpp` separately on Android phone and then integrate it with `llama-cpp-python`.
   2. It's important to note that `llama-cpp-python` serves as a Python wrapper around the `llama.cpp` library. This option allows you to customize and replace the underlying `llama.cpp` implementation to suit your specific needs.

### Option 1: Building from Ubuntu/Mac Laptop

#### Install Android NDK

Run these commands first:

```bash
cd ~ && \
wget https://dl.google.com/android/repository/commandlinetools-linux-8512546_latest.zip && \ 
unzip commandlinetools-linux-8512546_latest.zip && \ 
mkdir -p ~/android-sdk/cmdline-tools && \ 
mv cmdline-tools latest && \ 
mv latest ~/android-sdk/cmdline-tools/ && \ 
rm -rf commandlinetools-linux-8512546_latest.zip 
```

Check your OpenJDK location (please making sure that you download the OpenJDK-21 )

```bash
readlink -f $(which java)
```

Set `JAVA_HOME`:

```bash
vim ~/.bashsrc
```

```bash
export JAVA_HOME=/your/openjdk/directory
```

```bash
source ~/.bashrc
```

Finally, run this:

```bash
yes | ~/android-sdk/cmdline-tools/latest/bin/sdkmanager "ndk;26.3.11579264" 
```

#### Install OpenCL headers and ICD loader

```bash
# You can choose your own directory, here's an example from Qualcomm's official tutorial:
mkdir -p ~/dev/llm && cd ~/dev/llm
```

```bash
git clone https://github.com/KhronosGroup/OpenCL-Headers
```

```bash
cd OpenCL-Headers && cp -r CL ~/android-sdk/ndk/26.3.11579264/toolchains/llvm/prebuilt/linux-x86_64/sysroot/usr/include 
```

For MacOS, it will be:
```
~/android-sdk/ndk/26.3.11579264/toolchains/llvm/prebuilt/darwin-x86_64/sysroot/usr/include 
```

Then:

```bash
cd ~/dev/llm 
```

```bash
# If you are using MacOS, you may need to change your OPENCL_ICD_LOADER_HEADERS_DIR
git clone https://github.com/KhronosGroup/OpenCL-ICD-Loader && \ 
cd OpenCL-ICD-Loader && \ 
mkdir build_ndk26 && cd build_ndk26 && \ 
cmake .. -G Ninja -DCMAKE_BUILD_TYPE=Release \ 
  -DCMAKE_TOOLCHAIN_FILE=~/android-sdk/ndk/26.3.11579264/build/cmake/android.toolchain.cmake \ 
  -DOPENCL_ICD_LOADER_HEADERS_DIR=~/android-sdk/ndk/26.3.11579264/toolchains/llvm/prebuilt/linux-x86_64/sysroot/usr/include \ 
  -DANDROID_ABI=arm64-v8a \ 
  -DANDROID_PLATFORM=24 \ 
  -DANDROID_STL=c++_shared
```

```bash
ninja && cp libOpenCL.so ~/android-sdk/ndk/26.3.11579264/toolchains/llvm/prebuilt/linux-x86_64/sysroot/usr/lib/aarch64-linux-android 
```

#### Build llama.cpp with the Adreno OpenCL backend

```bash
cd ~/dev/llm 
```

```bash
git clone https://github.com/ggerganov/llama.cpp && \
cd llama.cpp && \ 
mkdir build-android && cd build-android 
```

```bash
cmake .. -G Ninja \ 
  -DCMAKE_TOOLCHAIN_FILE=$HOME/android-sdk/ndk/26.3.11579264/build/cmake/android.toolchain.cmake \ 
  -DANDROID_ABI=arm64-v8a \ 
  -DANDROID_PLATFORM=android-28 \ 
  -DBUILD_SHARED_LIBS=OFF \ 
  -DGGML_OPENCL=ON
```

```bash
ninja
```

Finally, copy the `dev` folder into the termux

-  Making sure you run `termux-setup-storage` on termux



Now, the executive files is available under `llama.cpp/build-android/bin`

Run `llama-bench` to test:

```bash
./llama-bench -m /your/model/directory -ngl 99
```

Result should looks like this:

```
ggml_opencl: selecting platform: 'QUALCOMM Snapdragon(TM)'
ggml_opencl: selecting device: 'QUALCOMM Adreno(TM) 830'
ggml_opencl: OpenCL driver: OpenCL 3.0 QUALCOMM build: commit unknown Compiler E031.47.18.21
ggml_opencl: vector subgroup broadcast support: true
ggml_opencl: device FP16 support: true
ggml_opencl: mem base addr align: 128
ggml_opencl: max mem alloc size: 1024 MB
ggml_opencl: SVM coarse grain buffer support: true
ggml_opencl: SVM fine grain buffer support: true
ggml_opencl: SVM fine grain system support: false
ggml_opencl: SVM atomics support: true
ggml_opencl: flattening quantized weights representation as struct of arrays (GGML_OPENCL_SOA_Q)
ggml_opencl: using kernels optimized for Adreno (GGML_OPENCL_USE_ADRENO_KERNELS)
...
```



### Option 2: Building from Android Phone (Recommend)

**Note: As of 04/27/2025, building from newest version of llama.cpp will cause segmentation fault. Please select a version before *b5028*. For more details, please visit: https://github.com/ggml-org/llama.cpp/issues/9289#issuecomment-2788276223**



Clone llama.cpp source code:

```bash
git clone https://github.com/ggml-org/llama.cpp.git
```

```bash
cd llama.cpp
```

If the segmentation fault is not fixed, then we need to switch to older version:

```bash
git reset --hard b5026
```

Note: set `BUILD_SHARED_LIBS` to `ON` if you want to use `llama-cpp-python` later:

```bash
cmake -B build-android \
-DBUILD_SHARED_LIBS=ON \
-DGGML_OPENCL=ON \
-DGGML_OPENCL_EMBED_KERNELS=ON \
-DGGML_OPENCL_USE_ADRENO_KERNELS=ON
```

Build the llama.cpp:

```bash
cmake --build build-android --config Release
```

Same as previous section, test by:

```bash
./llama-bench -m /your/model/directory -ngl 99
```



## `llama-cpp-python` with Adreno OpenCL Backend

**Note: Making sure you have set `BUILD_SHARED_LIBS=ON` when building the `llama.cpp`**

First, you need to set the eviromental variable:

```bash
vim ~/.bashrc
```

Export the following path:

```bash
# The path of libllama.so is under llama.cpp/build-android/bin
export LLAMA_CPP_LIB_PATH=/home/.../llama.cpp/build-android/bin
```

```bash
source ~/.bashrc
```

Now, install llama-cpp-python with `LLAMA_BUILD=OFF`

```bash
CMAKE_ARGS="-DLLAMA_BUILD=OFF" pip install llama-cpp-python --force-reinstall
```

To test it, run:

```bash
python -c "from llama_cpp import Llama; llm = Llama(model_path="your/model/path", n_gpu_layers=99, verbose=True)"
```

If you return verbose information like this:
```bash
...
load_tensors: offloading 28 repeating layers to GPU
load_tensors: offloading output layer to GPU
load_tensors: offloaded 29/29 layers to GPU
load_tensors:   CPU_Mapped model buffer size =   434.29 MiB
load_tensors:       OpenCL model buffer size =  1780.40 MiB
...
```

This means that `llama-cpp-python` is using the custom-built `llama.cpp`

## Testing Results

Device: OnePlus 13

|        model        |   size   | params | backend | ngl  | test  | t/s            |
| :-----------------: | :------: | :----: | :-----: | :--: | :---: | -------------- |
|  llama 3.2 3B Q4_0  | 1.78 GiB | 3.21 B | OpenCL  |  99  | pp512 | 168.47 ± 0.98  |
|  llama 3.2 3B Q4_0  | 1.78 GiB | 3.21 B | OpenCL  |  99  | tg128 | 17.57 ± 1.40   |
|  llama 3.2 3B Q4_0  | 1.78 GiB | 3.21 B | OpenCL  |  99  | tg256 | 15.45 ± 0.59   |
|  llama 3.2 3B Q4_0  | 1.78 GiB | 3.21 B | OpenCL  |  99  | tg512 | 13.60 ± 0.79   |
|   gemma 2 2B Q4_0   | 1.51 GiB | 2.61 B | OpenCL  |  99  | pp512 | 187.26 ± 0.49  |
|   gemma 2 2B Q4_0   | 1.51 GiB | 2.61 B | OpenCL  |  99  | tg128 | 8.11 ± 0.13    |
|   gemma 2 2B Q4_0   | 1.51 GiB | 2.61 B | OpenCL  |  99  | tg256 | 8.06 ± 0.07    |
|   gemma 2 2B Q4_0   | 1.51 GiB | 2.61 B | OpenCL  |  99  | tg512 | 8.02 ± 0.11    |
| phi 3 3.5 mini Q4_0 | 2.03 GiB | 3.82 B | OpenCL  |  99  | pp512 | 110.60 ± 11.96 |
| phi 3 3.5 mini Q4_0 | 2.03 GiB | 3.82 B | OpenCL  |  99  | tg128 | 16.76 ± 0.37   |
| phi 3 3.5 mini Q4_0 | 2.03 GiB | 3.82 B | OpenCL  |  99  | tg256 | 16.10 ± 0.53   |
| phi 3 3.5 mini Q4_0 | 2.03 GiB | 3.82 B | OpenCL  |  99  | tg512 | 14.89 ± 0.16   |
