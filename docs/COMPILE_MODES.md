# 四种编译方式经验总结

本文总结了 hwinfo 四种不同编译输出方式的完整步骤，包含遇到的问题和解决方案。

## 四种编译方式对比

| 方式 | 输出 | 平台 | 链接方式 | 适用场景 |
|------|------|--------|----------|----------|
| 1. Windows 动态 | `build/Release/*.dll` | Windows | 动态链接 | 需要 DLL 分离，方便更新 |
| 2. Windows 静态 | `build_static/src/Release/*.lib` | Windows | 静态链接 | 单 EXE 部署，推荐 |
| 3. Linux (WSL2) 动态 | `build_linux/lib*.so` | Linux | 动态链接 | 需要 `.so` 分离 |
| 4. Linux (WSL2) 静态 | `build_linux_static/src/lib*.a` | Linux | 静态链接 | 单二进制部署，推荐 |

---

## 一、Windows 动态编译（原默认）

### 编译步骤

```bash
# 1. 创建构建目录
mkdir build
cd build

# 2. CMake 配置
cmake -B . -DCMAKE_BUILD_TYPE=Release -DCMAKE_CONFIGURATION_TYPES=Release ..

# 3. 编译
cmake --build . --config Release
```

### 输出位置

```
build/Release/
├── hwinfo.dll              # 总的动态库
├── hwinfo_battery.dll      # 分模块动态库
├── hwinfo_cpu.dll
├── hwinfo_disk.dll
├── hwinfo_gpu.dll
├── hwinfo_mainboard.dll
├── hwinfo_network.dll
├── hwinfo_os.dll
├── hwinfo_ram.dll
├── system_info.exe
└── live_monitor.exe
```

### 问题解决

**问题**：`warning C4819: 该文件包含不能在当前代码页(936)中表示的字符` → 错误

**解决**：在 `CMakeLists.txt` 的 MSVC 编译选项添加 `/wd4819`：

```cmake
if(MSVC)
    add_compile_options(/W4 /WX /wd4819)
endif()
```

---

## 二、Windows 静态编译

### 编译步骤

```bash
# 1. 创建构建目录
mkdir build_static
cd build_static

# 2. CMake 配置（开启静态，关闭动态）
cmake -B . -DHWINFO_STATIC=ON -DHWINFO_SHARED=OFF -DCMAKE_BUILD_TYPE=Release -DCMAKE_CONFIGURATION_TYPES=Release ..

# 3. 编译
cmake --build . --config Release
```

### 输出位置

```
build_static/src/Release/
├── hwinfo.lib              # 总的静态库
├── hwinfo_battery.lib      # 分模块静态库
├── hwinfo_cpu.lib
├── hwinfo_disk.lib
├── hwinfo_gpu.lib
├── hwinfo_mainboard.lib
├── hwinfo_network.lib
├── hwinfo_os.lib
└── hwinfo_ram.lib
```

### 问题解决

**问题**：`warning C4273: "hwinfo::Battery::Battery": dll 链接不一致` → 错误

**原因分析**：
- 头文件中 `HWINFO_API` 宏根据 `HWINFO_EXPORTS` 决定是 `__declspec(dllexport)` 还是 `__declspec(dllimport)`
- 静态编译时，既没有 `HWINFO_EXPORTS` 也没有 `HWINFO_STATIC`，所以默认是 `__declspec(dllimport)`
- 但我们实际上是静态链接，导致声明不一致，产生警告，因为 `/WX` 所以警告变错误

**解决**：两步修改：

1. **修改 `include/hwinfo/platform.h`** 添加 `HWINFO_STATIC` 判断：

```c
// dll exports/imports for windows shared libraries
#ifdef _WIN32
#ifdef HWINFO_EXPORTS
#define HWINFO_API __declspec(dllexport)
#elif defined(HWINFO_STATIC)
#define HWINFO_API
#else
#define HWINFO_API __declspec(dllimport)
#endif
#pragma warning(disable : 4251)
#else
#define HWINFO_API __attribute__((visibility("default")))
#endif
```

2. **修改 `src/CMakeLists.txt`** 给每个目标添加定义：

```cmake
if(${HWINFO_STATIC})
    target_compile_definitions(hwinfo_battery PUBLIC -DHWINFO_STATIC)
endif()
```

对所有组件（battery、cpu、disk、gpu、mainboard、os、ram、network）以及 combined 都添加这个定义。

**经验总结**：这个问题只在 Windows 静态编译时出现，Linux 不会有这个问题。

---

## 三、Linux (WSL2) 动态编译

### 编译步骤

在 WSL2 终端中执行：

```bash
# 1. 进入项目目录（WSL2 可以直接访问 Windows 文件）
cd /mnt/f/Project/hwinfo

# 2. 创建构建目录
mkdir -p build_linux
cd build_linux

# 3. CMake 配置
cmake -DCMAKE_BUILD_TYPE=Release ..

# 4. 编译（使用所有核心并行编译）
make -j$(nproc)
```

### 输出位置

```
build_linux/
├── libhwinfo.so              # 总的共享库
├── libhwinfo_battery.so      # 分模块共享库
├── libhwinfo_cpu.so
├── libhwinfo_disk.so
├── libhwinfo_gpu.so
├── libhwinfo_mainboard.so
├── libhwinfo_network.so
├── libhwinfo_os.so
├── libhwinfo_ram.so
├── system_info               # 示例程序
└── live_monitor              # 示例程序
```

### 问题解决

**问题**：`Clock skew detected.  Your build may be incomplete` 警告

**原因**：WSL2 访问 Windows NTFS 文件系统（`/mnt/f/...`）时，Windows 和 WSL2 的时钟不同步

**解决**：这只是警告，不影响使用，可以安全忽略。如果想彻底解决，可以把源码复制到 WSL2 原生文件系统（`/home/xxx/...`）再编译。

---

## 四、Linux (WSL2) 静态编译

### 编译步骤

在 WSL2 终端中执行：

```bash
# 1. 进入项目目录
cd /mnt/f/Project/hwinfo

# 2. 创建构建目录
mkdir -p build_linux_static
cd build_linux_static

# 3. CMake 配置（开启静态，关闭动态）
cmake -DCMAKE_BUILD_TYPE=Release -DHWINFO_STATIC=ON -DHWINFO_SHARED=OFF ..

# 4. 编译
make -j$(nproc)
```

### 输出位置

```
build_linux_static/src/
├── libhwinfo.a              # 总的静态库
├── libhwinfo_battery.a      # 分模块静态库
├── libhwinfo_cpu.a
├── libhwinfo_disk.a
├── libhwinfo_gpu.a
├── libhwinfo_mainboard.a
├── libhwinfo_network.a
├── libhwinfo_os.a
├── libhwinfo_ram.a
├── ../system_info           # 示例程序
└── ../live_monitor          # 示例程序
```

### 问题解决

在 Linux 静态编译没有遇到特殊问题，我们修复的 Windows dll 链接不一致问题不影响 Linux，因为：
- Linux 不需要 `__declspec(dllexport)/dllimport`
- 我们的修复只对 `_WIN32` 有效，对 Linux 没有影响

---

## 四种编译方式目录结构

```
hwinfo/
├── build/                # Windows 动态
│   └── Release/
│       └── *.dll
├── build_static/          # Windows 静态
│   └── src/Release/
│       └── *.lib
├── build_linux/          # Linux 动态 (WSL2)
│   └── *.so
└── build_linux_static/   # Linux 静态 (WSL2)
    └── src/
        └── *.a
```

---

## 快速命令备忘

### Windows 动态
```powershell
cd f:\Project\hwinfo
mkdir build
cd build
cmake -B . -DCMAKE_BUILD_TYPE=Release -DCMAKE_CONFIGURATION_TYPES=Release ..
cmake --build . --config Release
```

### Windows 静态
```powershell
cd f:\Project\hwinfo
mkdir build_static
cd build_static
cmake -B . -DHWINFO_STATIC=ON -DHWINFO_SHARED=OFF -DCMAKE_BUILD_TYPE=Release -DCMAKE_CONFIGURATION_TYPES=Release ..
cmake --build . --config Release
```

### Linux 动态 (WSL2)
```bash
cd /mnt/f/Project/hwinfo
mkdir -p build_linux
cd build_linux
cmake -DCMAKE_BUILD_TYPE=Release ..
make -j$(nproc)
```

### Linux 静态 (WSL2)
```bash
cd /mnt/f/Project/hwinfo
mkdir -p build_linux_static
cd build_linux_static
cmake -DCMAKE_BUILD_TYPE=Release -DHWINFO_STATIC=ON -DHWINFO_SHARED=OFF ..
make -j$(nproc)
```

---

## 如何在其他项目中使用

### CMakeLists.txt 配置示例

```cmake
# 设置 hwinfo 路径
set(HWINFO_ROOT ../hwinfo)

# 添加头文件目录
include_directories(${HWINFO_ROOT}/include)

# 根据平台选择静态库路径
if(WIN32)
    link_directories(${HWINFO_ROOT}/build_static/src/Release)
    target_link_libraries(your_project PRIVATE hwinfo)
else()
    link_directories(${HWINFO_ROOT}/build_linux_static/src)
    target_link_libraries(your_project PRIVATE hwinfo)
endif()
```

### 使用头文件

```cpp
#include <hwinfo/hwinfo.h>

// 获取所有CPU
std::vector<hwinfo::CPU> cpus = hwinfo::getAllCPUs();

// 获取内存信息
hwinfo::Memory memory;
std::cout << "Total RAM: " << memory.size() << " bytes" << std::endl;

// 获取所有磁盘
std::vector<hwinfo::Disk> disks = hwinfo::getAllDisks();

// 获取GPU
std::vector<hwinfo::GPU> gpus = hwinfo::getAllGPUs();

// 获取主板
hwinfo::MainBoard mainboard = hwinfo::getMainBoard();

// 获取操作系统信息
hwinfo::OS os;
```

---

## 常见问题汇总

| 问题 | 原因 | 解决方法 |
|------|------|----------|
| `warning C4819` 编码警告 → 错误 | 文件编码在当前代码页无法表示 | 添加 `/wd4819` 编译选项 |
| `warning C4273` dll 链接不一致 → 错误 | 静态编译时 `HWINFO_API` 仍然是 `dllimport` | 添加 `HWINFO_STATIC` 宏处理 |
| `Clock skew detected` (WSL2) | Windows/WSL2 时钟不同步 | 忽略警告，不影响使用 |
| `cmake 不是内部或外部命令` | CMake 未安装或未添加到 PATH | 重新安装 CMake，勾选添加 PATH |
| `无法打开包括文件: 'iostream'` (Windows) | Visual Studio 未安装 C++ 工具 | 运行 VS 安装程序，勾选"使用C++的桌面开发" |

---

## 本次修改总结

为了支持静态编译，我们做了以下修改：

1. **[include/hwinfo/platform.h](file:///f:/Project/hwinfo/include/hwinfo/platform.h)** - 添加 `HWINFO_STATIC` 宏判断
2. **[src/CMakeLists.txt](file:///f:/Project/hwinfo/src/CMakeLists.txt)** - 给所有目标添加 `HWINFO_STATIC` 编译定义
3. **[CMakeLists.txt](file:///f:/Project/hwinfo/CMakeLists.txt)** - 已经添加 `/wd4819` 解决编码警告

这些修改：
- ✅ 不影响原有的动态编译
- ✅ 只在 `HWINFO_STATIC=ON` 时生效
- ✅ 保持项目原有架构不变
- ✅ 符合"不想项目变更太多"的要求

---

## 验证编译结果

编译完成后，可以运行示例程序验证：

### Windows
```powershell
cd build_static\Release
.\system_info.exe
```

### Linux (WSL2)
```bash
cd build_linux_static
./system_info
```

如果能正常输出硬件信息，说明编译成功！🎉
