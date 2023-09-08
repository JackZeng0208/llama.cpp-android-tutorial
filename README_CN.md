# 在安卓手机上运行llama.cpp（基于termux)

[English](README.md)

llama.cpp 链接： https://github.com/ggerganov/llama.cpp

相似教程：https://ivonblog.com/posts/alpaca-cpp-termux-android/#21-%E7%9B%B4%E6%8E%A5%E5%9C%A8termux%E8%B7%91

## Termux下载

官网：[termux](https://termux.dev/cn/index.html)

国内用户在下载完成后可以使用如下命令修改源：

```bash
termux-change-repo
```

选择清华镜像源即可

更多信息请参考：[Termux镜像使用帮助](https://mirrors.tuna.tsinghua.edu.cn/help/termux/)

## 下载源代码及必要程序

在termux内下载必要工具：

```bash
pkg install clang wget git cmake
```

获取源代码：

```bash
git clone https://github.com/ggerganov/llama.cpp.git
```


## 导入模型

在导入模型前需在termux端输入`termux-setup-storage` ， 在给予存储权限后termux会在`$HOME`下创建`storage`文件夹。通过此文件夹即可访问安卓系统内其他文件。具体可参考：https://wiki.termux.com/wiki/Termux-setup-storage

从电脑端用`adb push`即可导入：

```
adb push \path\to\your\model\on\windows /storage/emulated/0/download
```

termux的`~/storage/downloads`与手机的下载文件夹是共享的。将模型移动至`~/llama.cpp/models`即可：

```bash
mv ~/storage/downloads/model_name ~/llama.cpp/models
```

## 编译和运行

**强烈推荐使用cmake!**

### 基于Android NDK（非OpenCL）

#### 下载Pre-build NDK

地址：https://github.com/lzhiyong/termux-ndk/releases/tag/ndk-r23/

```bash
wget https://github.com/lzhiyong/termux-ndk/releases/download/ndk-r23/android-ndk-r23c-aarch64.zip
```

解压并设置PATH（默认存储在termux `~`下）：

```bash
unzip YOUR_ANDROID_NDK_ZIP_FILE
export NDK=~/path/to/your/unzip/directory
```

#### 构建

在`~/llama.cpp/build`内构建：

```bash
mkdir build
cd build
cmake -DCMAKE_TOOLCHAIN_FILE=$NDK/build/cmake/android.toolchain.cmake -DANDROID_ABI=arm64-v8a -DANDROID_PLATFORM=android-23 -DCMAKE_C_FLAGS=-march=armv8.4a+dotprod ..
make
```

运行：

```bash
cd bin/
./main YOUR_PARAMETERS
```

### 基于OpenCL + CLBlast（推荐）

下载必要程序：

```bash
apt install ocl-icd opencl-headers opencl-clhpp clinfo libopenblas
```

手动编译CLBlast并将`clblast.h`放入llama.cpp中：

```bash
git clone https://github.com/CNugteren/CLBlast.git
cd CLBlast
cmake .
make
cp libclblast.so* $PREFIX/lib
cp ./include/clblast.h ../llama.cpp
```

将OpenBLAS的必要文件复制到llama.cpp文件夹内：

```bash
cp /data/data/com.termux/files/usr/include/openblas/cblas.h .
cp /data/data/com.termux/files/usr/include/openblas/openblas_config.h .
```

#### 构建

```bash
cd ~/llama.cpp
mkdir build
cd build
cmake .. -DLLAMA_CLBLAST=ON
cmake --build . --config Release
```

在`~/.bashrc`内添加`LD_LIBRARY_PATH`，避免重启termux后需要重复操作（让llama.cpp直接运行在本机GPU上）：

```bash
echo "export LD_LIBRARY_PATH=/vendor/lib64:$LD_LIBRARY_PATH" >> ~/.bashrc
```

检查OpenCL是否识别到GPU：

```bash
clinfo -l
```

以高通平台为例，如果配置成功则会显示：

```bash
Platform #0: QUALCOMM Snapdragon(TM)
 `-- Device #0: QUALCOMM Adreno(TM)
```

运行：
```bash
cd bin/
./main YOUR_PARAMETERS
```

#### 效果

- SoC：骁龙 8 Gen 2
- RAM：16 GB
- 模型：llama-2-7B-Chat-Q4_0.gguf（下载：[Rabinovich/Llama-2-7B-Chat-GGUF · Hugging Face](https://huggingface.co/Rabinovich/Llama-2-7B-Chat-GGUF)）
- 运行多轮长对话
- 参数：
  - Context size = 4096
  - Batch size = 16
  - Threads = 4

结果：

- Load time = 1129.17 ms
- 3.67 tokens per second
