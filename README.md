# Vulkan Triangle Tutorial

学习 Vulkan 的第一个三角形项目，使用 vcpkg 管理依赖。

## 项目结构

```
vk/
├── vcpkg.json              # vcpkg 依赖配置
├── CMakePresets.json       # CMake 预设配置
├── CMakeLists.txt          # CMake 构建配置
├── src/
│   └── main.cpp            # Vulkan 三角形应用代码
├── vcpkg_installed/        # vcpkg 本地安装的依赖
└── build/                  # 构建输出目录
```

## 依赖

- **vulkan-headers**: Vulkan API 头文件
- **vulkan-loader**: Vulkan 加载器
- **vulkan-hpp**: Vulkan C++ 绑定
- **glfw3**: 窗口和输入管理
- **glm**: 数学库
- **stb**: 图像加载库

## 环境要求

### Windows
- Windows 10/11
- Visual Studio 2019/2022 或 Build Tools
- CMake 3.20+
- vcpkg

### macOS
- macOS 10.15+ (支持 Apple Silicon 和 Intel)
- Xcode Command Line Tools
- CMake 3.20+
- vcpkg

## 编译步骤

### Windows

#### 1. 配置项目（自动安装依赖）
```bash
# 使用 Windows 专用预设
cmake --preset=vcpkg-windows

# 或使用自动检测预设
cmake --preset=vcpkg
```

#### 2. 生成编译数据库（用于 clangd）
```bash
cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=1 --preset=vcpkg-windows
```

#### 3. 编译项目
```bash
cmake --build build
```

#### 4. 运行程序
```bash
./build/VulkanTriangle.exe
```

### macOS

#### 1. 配置项目（自动安装依赖）
```bash
# 使用 macOS 专用预设
cmake --preset=vcpkg-macos

# 或使用自动检测预设
cmake --preset=vcpkg
```

#### 2. 生成编译数据库（用于 clangd）
```bash
cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=1 --preset=vcpkg-macos
```

#### 3. 编译项目
```bash
cmake --build build
```

#### 4. 运行程序
```bash
./build/VulkanTriangle
```

## 调试配置

### Windows

#### 1. 使用 VS Code 调试
由于 Cursor 的 C++ 扩展不支持 `cppvsdbg` 调试器，建议使用 VS Code 进行调试：

```bash
# 在 MSVC 环境中启动 VS Code
code .
```

#### 2. 调试配置
项目已配置 `.vscode/launch.json` 和 `.vscode/tasks.json`：
- 支持 Debug 和 Release 配置
- 自动编译和启动调试
- 配置环境变量和路径

### macOS

#### 1. 使用 LLDB 调试
macOS 上可以使用 LLDB 进行调试：

```bash
# 使用 LLDB 启动程序
lldb ./build/VulkanTriangle

# 在 LLDB 中设置断点并运行
(lldb) breakpoint set --name main
(lldb) run
```

#### 2. 使用 Xcode 调试
也可以使用 Xcode 进行图形化调试：

```bash
# 生成 Xcode 项目
cmake --preset=vcpkg-macos -G Xcode

# 在 Xcode 中打开项目
open VulkanTriangle.xcodeproj
```

## 功能特性

- ✅ 创建 Vulkan 实例
- ✅ 选择物理设备
- ✅ 创建设备和队列
- ✅ 窗口管理 (GLFW)
- ✅ 基础错误处理
- ✅ 渲染管线
- ✅ 绘制三角形
- ✅ 交换链管理
- ✅ 命令缓冲区
- ✅ 同步对象

## 开发环境

- 使用 vcpkg 进行本地依赖管理
- CMakePresets 配置构建环境
- 跨平台支持：Windows (Visual Studio) 和 macOS (Xcode/Unix Makefiles)
- clangd 作为代码补全和跳转工具

## 开发环境配置

### Windows

#### ⚠️ 重要：x64 编译环境
**必须先打开 Visual Studio x64 命令提示符**，否则会出现架构冲突错误：
```
库计算机类型"x64"与目标计算机类型"x86"冲突
```

#### 1. 启动开发环境
为了正确识别 MSVC 编译器和避免架构冲突，需要从 MSVC x64 命令提示符启动：

```bash
# 方法1：直接运行 x64 命令提示符
"C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\VC\Auxiliary\Build\vcvars64.bat"

# 方法2：打开 Visual Studio Developer Command Prompt
# 在开始菜单中找到 "Developer Command Prompt for VS 2022" 
# 或 "x64 Native Tools Command Prompt for VS 2022"

# 在 MSVC 环境中启动 Cursor
cursor .
```

#### 2. 验证 x64 环境
在命令提示符中验证编译环境：
```bash
# 检查编译器架构
cl  # 应该显示 x64 相关信息
```

### macOS

#### 1. 安装 Xcode Command Line Tools
```bash
# 安装 Xcode Command Line Tools
xcode-select --install

# 验证安装
clang --version
```

#### 2. 配置 vcpkg
```bash
# 设置 vcpkg 环境变量
export VCPKG_ROOT=/path/to/vcpkg
export PATH=$VCPKG_ROOT:$PATH
```

### 通用配置

#### 1. clangd 配置
项目已配置使用 clangd 作为代码补全工具：
- 自动读取 `build/compile_commands.json`
- 支持 Vulkan 头文件跳转
- 提供准确的代码补全

#### 2. 开发与调试环境
- **开发环境**：使用 Cursor 进行代码编写
  - clangd 提供代码补全和跳转
  - 支持 Vulkan 头文件智能提示
- **调试环境**：
  - **Windows**: 使用 VS Code 进行调试（Cursor 不支持 `cppvsdbg` 调试器）
  - **macOS**: 使用 LLDB 或 Xcode 进行调试

## 跨平台支持

### CMake 预设配置
项目提供了三个 CMake 预设：

- **`vcpkg-macos`**: macOS 专用配置
  - 使用 `arm64-osx` triplet
  - 使用 `Unix Makefiles` 生成器
- **`vcpkg-windows`**: Windows 专用配置
  - 使用 `x64-windows` triplet
  - 使用 `Ninja` 生成器
- **`vcpkg`**: 自动检测配置
  - 不指定 triplet，由 vcpkg 自动选择
  - 使用 `Unix Makefiles` 生成器

### 平台特定功能
- **Windows**: 支持 Visual Studio 和 Ninja 构建
- **macOS**: 支持 Xcode 和 Unix Makefiles 构建
- **自动平台检测**: CMakeLists.txt 自动检测平台并设置相应的 Vulkan 平台定义

## 参考资料

- [vcpkg - C++ Library Manager](https://github.com/microsoft/vcpkg) - Microsoft 维护的 C++ 包管理器
- [Vulkan Tutorial](https://docs.vulkan.org/tutorial/latest/00_Introduction.html) - Khronos 官方 Vulkan 教程
- [CMake Presets](https://cmake.org/cmake/help/latest/manual/cmake-presets.7.html) - CMake 预设配置文档

## TODO
- [GFXReconstruct](https://www.lunarg.com/mastering-gfxreconstuct-part-1/) - TODO
- [Queue](https://docs.vulkan.org/guide/latest/queues.html) - Queue的概念和map
