# hwinfo 编译教程

本教程面向**完全没有编译经验**的读者，循序渐进地教你如何在不同平台下编译 hwinfo 库。

## 前置知识

什么是编译？

> 我们写的 C++ 源代码是人类能读懂的文本文件，**编译**就是把这些源代码转换成计算机能直接运行的二进制文件（`.dll`/`.so`/`.exe`）的过程。

hwinfo 是什么？

> hwinfo 是一个开源的 C++ 硬件信息获取库，可以读取 CPU、内存、磁盘、GPU、主板等硬件信息。

## 准备工作

无论在哪个平台编译，你都需要先安装这些工具：

1. **Git** - 用来下载源代码
2. **CMake** - 用来自动生成编译配置
3. **C++ 编译器** - 用来实际编译源代码

下面分不同平台说明具体安装方法。

---

## 一、Windows 平台编译

### 1. 安装工具

#### 安装 Git
- 下载地址：https://git-scm.com/download/win
- 一路下一步安装即可

#### 安装 CMake
- 下载地址：https://cmake.org/download/
- 选择 Windows 版本，下载 `.msi` 安装包
- 安装时勾选"Add CMake to the system PATH"

#### 安装 Visual Studio
- 下载地址：https://visualstudio.microsoft.com/zh-hans/downloads/
- 选择 **Community** 版本（免费）
- 安装时勾选 **"使用C++的桌面开发"** 工作负载
- 安装完成后重启电脑

### 2. 获取源代码

打开命令提示符（CMD）或 PowerShell，执行：

```bash
# 克隆源代码到当前目录
git clone https://github.com/你的用户名/hwinfo.git

# 进入项目目录
cd hwinfo
```

### 3. 编译

在项目根目录执行：

```bash
# 创建 build 目录（放编译生成的文件）
mkdir build

# 进入 build 目录
cd build

# 运行 CMake 配置
cmake -B . -DCMAKE_BUILD_TYPE=Release -DCMAKE_CONFIGURATION_TYPES=Release ..

# 开始编译
cmake --build . --config Release
```

### 4. 编译结果

编译完成后，打开 `build\Release\` 目录，你会看到：

```
build/Release/
├── hwinfo.dll              # 总的动态链接库（推荐使用）
├── hwinfo_battery.dll      # 电池信息（分模块）
├── hwinfo_cpu.dll          # CPU 信息（分模块）
├── hwinfo_disk.dll         # 磁盘信息（分模块）
├── hwinfo_gpu.dll          # GPU 信息（分模块）
├── hwinfo_mainboard.dll    # 主板信息（分模块）
├── hwinfo_network.dll      # 网络信息（分模块）
├── hwinfo_os.dll           # 操作系统信息（分模块）
├── hwinfo_ram.dll          # 内存信息（分模块）
├── system_info.exe         # 示例程序：显示系统信息
└── live_monitor.exe        # 示例程序：实时监控
```

### 5. 测试运行

在 `build\Release\` 目录双击 `system_info.exe` 运行，你应该能看到类似这样的输出：

```
Hardware Report:
----------------------------------- CPU ------------------------------------
Socket 0:
 vendor:            GenuineIntel
 model:             13th Gen Intel(R) Core(TM) i7-13700H
 physical cores:    10
 logical cores:     20
 ...
```

如果能正常输出，说明编译成功！🎉

---

## 二、Linux 平台编译（含 WSL2）

### 1. 什么是 WSL2？

> WSL2 是 Windows 下的 Linux 子系统，可以在 Windows 中直接运行 Linux 程序。如果你想在 Windows 中获得 Linux 编译环境，推荐安装 WSL2。

**安装 WSL2（Windows 10/11）：**

以管理员身份打开 PowerShell，执行：

```powershell
wsl --install
```

安装完成后重启电脑，按照提示设置用户名和密码即可。

### 2. 安装工具

打开 WSL2 终端（或原生 Linux 终端），执行：

```bash
# 更新包列表
sudo apt update

# 安装编译工具
sudo apt install -y build-essential cmake git
```

这个命令会自动安装：
- `build-essential` - 包含 gcc、g++、make 等编译工具
- `cmake` - CMake 配置工具
- `git` - Git 版本控制

### 3. 获取源代码

在 WSL2/Linux 终端执行：

```bash
# 进入 /mnt/f/ 就是 Windows 的 F 盘
cd /mnt/f/Project

# 克隆源代码
git clone https://github.com/你的用户名/hwinfo.git

# 进入项目目录
cd hwinfo
```

> 💡 WSL2 可以直接访问 Windows 的文件，路径格式是 `/mnt/c/...`、`/mnt/f/...`

### 4. 编译

```bash
# 创建 build_linux 目录
mkdir -p build_linux

# 进入 build_linux 目录
cd build_linux

# 运行 CMake 配置
cmake -DCMAKE_BUILD_TYPE=Release ..

# 开始编译（-j4 表示使用 4 线程并行编译，加快速度）
make -j4
```

### 5. 编译结果

编译完成后，`build_linux/` 目录下会有：

```
build_linux/
├── libhwinfo.so              # 总的共享库（推荐使用）
├── libhwinfo_battery.so      # 电池信息（分模块）
├── libhwinfo_cpu.so          # CPU 信息（分模块）
├── libhwinfo_disk.so         # 磁盘信息（分模块）
├── libhwinfo_gpu.so          # GPU 信息（分模块）
├── libhwinfo_mainboard.so    # 主板信息（分模块）
├── libhwinfo_network.so      # 网络信息（分模块）
├── libhwinfo_os.so           # 操作系统信息（分模块）
├── libhwinfo_ram.so          # 内存信息（分模块）
├── system_info               # 示例程序：显示系统信息
└── live_monitor              # 示例程序：实时监控
```

> 💡 Linux 下动态库后缀是 `.so`（shared object），Windows 下是 `.dll`（dynamic link library），作用是一样的。

### 6. 测试运行

```bash
# 运行示例程序
./system_info
```

你应该能看到硬件信息输出，说明编译成功！🎉

---

## 三、macOS 平台编译

### 1. 安装工具

首先安装 **Xcode Command Line Tools**：

```bash
xcode-select --install
```

在弹出的窗口中点击"安装"即可。这会自动安装 gcc、clang、make 等工具。

然后安装 Homebrew（macOS 包管理器）：

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

安装完成后，安装 CMake：

```bash
brew install cmake git
```

### 2. 获取源代码

```bash
git clone https://github.com/你的用户名/hwinfo.git
cd hwinfo
```

### 3. 编译

```bash
mkdir -p build_macos
cd build_macos
cmake -DCMAKE_BUILD_TYPE=Release ..
make -j$(sysctl -n hw.ncpu)
```

### 4. 测试运行

```bash
./system_info
```

---

## 四、编译选项说明

### 选择组件

hwinfo 默认开启所有组件。如果你不需要某些组件，可以在 CMake 配置时关闭：

```bash
# 比如不需要电池和网络，可以这样：
cmake -DHWINFO_BATTERY=OFF -DHWINFO_NETWORK=OFF ..
```

可用的选项：

| 选项 | 默认值 | 说明 |
|------|--------|------|
| `HWINFO_OS` | ON | 操作系统信息 |
| `HWINFO_MAINBOARD` | ON | 主板信息 |
| `HWINFO_CPU` | ON | CPU 信息 |
| `HWINFO_DISK` | ON | 磁盘信息 |
| `HWINFO_RAM` | ON | 内存信息 |
| `HWINFO_GPU` | ON | GPU 信息 |
| `HWINFO_GPU_OPENCL` | OFF | GPU 信息使用 OpenCL（需要额外依赖） |
| `HWINFO_BATTERY` | ON | 电池信息 |
| `HWINFO_NETWORK` | ON | 网络信息 |

### 静态库 vs 动态库

hwinfo 默认编译**动态库**（`.dll`/`.so`/`.dylib`）。

如果你想编译静态库，可以：

```bash
cmake -DHWINFO_SHARED=OFF -DHWINFO_STATIC=ON ..
```

| 类型 | Windows | Linux | macOS | 优点 | 缺点 |
|------|---------|-------|-------|------|------|
| 动态库 | `.dll` | `.so` | `.dylib` | 编译快，文件小 | 运行时需要带这些库文件 |
| 静态库 | `.lib` | `.a` | `.a` | 编译好后直接链接到你的程序 | 编译慢，文件大 |

### 编译示例程序

默认会编译示例程序。如果你不想编译示例，可以：

```bash
cmake -DBUILD_EXAMPLES=OFF ..
```

---

## 五、如何在你的项目中使用 hwinfo

### 方式一：单个库链接（推荐）

编译后会生成总的 `hwinfo.dll`（Windows）或 `libhwinfo.so`（Linux），包含所有启用的组件。使用起来最简单：

**CMakeLists.txt 示例：**
```cmake
find_package(hwinfo REQUIRED)
target_link_libraries(your_target PRIVATE lfreist-hwinfo::hwinfo_combined)
```

只需要部署一个库文件，简单方便。

### 方式二：分模块链接

如果你只需要部分组件，可以只链接需要的模块：

```cmake
find_package(hwinfo REQUIRED)
target_link_libraries(your_target PRIVATE
  lfreist-hwinfo::hwinfo_cpu
  lfreist-hwinfo::hwinfo_ram
  lfreist-hwinfo::hwinfo_disk
)
```

这样可以减小最终程序的大小。

### 方式三：作为 Git 子模块使用

如果你不想单独编译，可以把 hwinfo 作为子模块加到你的项目中：

```bash
# 在你的项目根目录执行
mkdir third_party
cd third_party
git clone https://github.com/你的用户名/hwinfo.git
```

然后在你的 `CMakeLists.txt` 中：

```cmake
# 如果你想关闭不需要的组件，先设置选项
set(HWINFO_BATTERY OFF CACHE BOOL "" FORCE)
set(HWINFO_NETWORK OFF CACHE BOOL "" FORCE)

# 添加子目录
add_subdirectory(third_party/hwinfo)

# 链接
target_link_libraries(your_target PRIVATE lfreist-hwinfo::hwinfo_combined)
```

---

## 六、常见问题解决

### Q1: 编译报错，提示 "error C2220: 警告被视为错误"

**A:** 这是因为文件编码问题，我们已经在 `CMakeLists.txt` 中添加了解决方案。如果你是从官方仓库拉取的代码，可以拉取我们修改后的版本：

```bash
git pull
# 重新编译即可
```

### Q2: WSL2 编译出现 "Clock skew detected" 警告

**A:** 这是正常的。因为 WSL2 访问 Windows 文件系统时，时钟同步可能有问题，不影响使用，可以忽略。

### Q3: 提示 "cmake 不是内部或外部命令"

**A:** CMake 没有安装，或者没有添加到 PATH 环境变量。重新安装 CMake，安装时记得勾选"Add CMake to the system PATH"。

### Q4: Windows 编译提示 "不开具 c 编译器" 或 "无法打开包括文件: 'iostream'"

**A:** Visual Studio 没有安装 C++ 开发工具。重新运行 Visual Studio 安装程序，勾选"使用C++的桌面开发"工作负载，然后安装。

### Q5: 我只想要一个总的 DLL/.so，不需要分模块，可以吗？

**A:** 可以。我们已经添加了总的库，你只需要拿 `hwinfo.dll`（Windows）或 `libhwinfo.so`（Linux）就可以了，其他分模块文件可以删除。

### Q6: 编译好的库可以在其他机器上使用吗？

**A:** 
- **Windows**: 编译好的 `hwinfo.dll` 可以在其他 Windows 机器上使用
- **Linux**: 编译好的 `libhwinfo.so` 可以在相同架构（x86_64）的 Linux 机器上使用
- 跨平台不能混用（Windows DLL 不能在 Linux 用，Linux .so 也不能在 Windows 用）

---

## 七、文件说明

编译完成后，哪些文件是你需要的？

### Windows
你需要：
- `include/` 头文件目录
- `build/Release/hwinfo.dll` - 动态库
- `build/Release/hwinfo.lib` - 导入库（编译你的程序时需要）

### Linux
你需要：
- `include/` 头文件目录
- `build_linux/libhwinfo.so` - 共享库

---

## 八、快速命令备忘

### Windows 完整编译流程

```powershell
git clone https://github.com/你的用户名/hwinfo.git
cd hwinfo
mkdir build
cd build
cmake -B . -DCMAKE_BUILD_TYPE=Release -DCMAKE_CONFIGURATION_TYPES=Release ..
cmake --build . --config Release
```

### Linux/WSL2 完整编译流程

```bash
git clone https://github.com/你的用户名/hwinfo.git
cd hwinfo
mkdir -p build_linux
cd build_linux
cmake -DCMAKE_BUILD_TYPE=Release ..
make -j$(nproc)
```

### macOS 完整编译流程

```bash
git clone https://github.com/你的用户名/hwinfo.git
cd hwinfo
mkdir -p build_macos
cd build_macos
cmake -DCMAKE_BUILD_TYPE=Release ..
make -j$(sysctl -n hw.ncpu)
```

---

## 九、总结

现在你已经学会了：

- ✅ 在 Windows 下编译 hwinfo
- ✅ 在 Linux/WSL2 下编译 hwinfo  
- ✅ 在 macOS 下编译 hwinfo
- ✅ 知道如何选择编译选项
- ✅ 知道如何在自己的项目中使用
- ✅ 会解决一些常见问题

如果你遇到其他问题，欢迎在 [GitHub Issues](https://github.com/laohuyou886/hwinfo/issues) 提问。

Happy coding! 🎉
