# llama.cpp Android Tutorial

[中文](README_CN.md)

llama.cpp link： https://github.com/ggerganov/llama.cpp

## Termux installation

Official Website: [termux](https://termux.dev/cn/index.html).

Change repo for faster speed:

```bash
termux-change-repo
```

Check [here](https://wiki.termux.com/wiki/Package_Management) for more help.

## Install necessary code and packages

Download following packages in termux:

```bash
pkg install clang wget git cmake
```

Obtain llama.cpp source code:

```bash
git clone https://github.com/ggerganov/llama.cpp.git
```


## Importing language model

Type`termux-setup-storage` in termux terminal before importing model. Grant access for termux so that user could access files outside of termux. For details, please visit: https://wiki.termux.com/wiki/Termux-setup-storage

Use `adb push` command to import:

```
adb push \path\to\your\model\on\windows /storage/emulated/0/download
```

`~/storage/downloads` in termux home directory shares download files on Android system. Move it to `~/llama.cpp/models` 

```bash
mv ~/storage/downloads/model_name ~/llama.cpp/models
```

## Compile and build

**Strongly recommend to use cmake rather than make**

### Based on Android NDK（Non-OpenCL）

#### Install Pre-build NDK

Location: https://github.com/lzhiyong/termux-ndk/releases/tag/ndk-r23/

```bash
wget https://github.com/lzhiyong/termux-ndk/releases/download/ndk-r23/android-ndk-r23c-aarch64.zip
```

Unzip and set NDK PATH:

```bash
unzip YOUR_ANDROID_NDK_ZIP_FILE
export NDK=~/path/to/your/unzip/directory
```

#### Build

Build under `~/llama.cpp/build`:

```bash
mkdir build
cd build
cmake -DCMAKE_TOOLCHAIN_FILE=$NDK/build/cmake/android.toolchain.cmake -DANDROID_ABI=arm64-v8a -DANDROID_PLATFORM=android-23 -DCMAKE_C_FLAGS=-march=armv8.4a+dotprod ..
make
```

Run:

```bash
cd bin/
./main YOUR_PARAMETERS
```

### Based on OpenCL + CLBlast（Recommend）

Download necessary packages: 

```bash
apt install ocl-icd opencl-headers opencl-clhpp clinfo libopenblas
```

Manually compile CLBlast and copy `clblast.h` into llama.cpp:

```bash
git clone https://github.com/CNugteren/CLBlast.git
cd CLBlast
cmake .
make
cp libclblast.so* $PREFIX/lib
cp ./include/clblast.h ../llama.cpp
```

Copy OpenBLAS files to llama.cpp:

```bash
cp /data/data/com.termux/files/usr/include/openblas/cblas.h .
cp /data/data/com.termux/files/usr/include/openblas/openblas_config.h .
```

#### Build

```bash
cd ~/llama.cpp
mkdir build
cd build
cmake .. -DLLAMA_CLBLAST=ON
cmake --build . --config Release
```

Add `LD_LIBRARY_PATH` under `~/.bashrc`（Run program directly on physical GPU）：

```bash
echo "export LD_LIBRARY_PATH=/vendor/lib64:$LD_LIBRARY_PATH" >> ~/.bashrc
```

Check GPU is available for OpenCL:

```bash
clinfo -l
```

If everything works fine, for Qualcomm Snapdragon SoC, it will display:

```bash
Platform #0: QUALCOMM Snapdragon(TM)
 `-- Device #0: QUALCOMM Adreno(TM)
```

Run:

```bash
cd bin/
./main YOUR_PARAMETERS
```

#### Results

- SoC: Qualcomm Snapdragon 8 Gen 2
- RAM: 16 GB
- Model: llama-2-7B-Chat-Q4_0.gguf（[Download](https://huggingface.co/Rabinovich/Llama-2-7B-Chat-GGUF)）
- Multiple long conversations
- Params:
  - Context size = 4096
  - Batch size = 16
  - Threads = 4

Result:

- Load time = 1129.17 ms
- 3.67 tokens per second