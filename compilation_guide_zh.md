# 编译与故障排除指南 - Godot Iroh

本文档总结了用于编译 Android 项目的命令、优化设置以及遇到的问题的解决方案。

## 1. Android 编译命令 (PowerShell)

要编译 Android 原生库 (`aarch64-linux-android`)，必须在运行 `cargo build` 之前配置 NDK 环境变量。

```powershell
# 配置 NDK 链接器和 clang 的环境变量
$env:CARGO_TARGET_AARCH64_LINUX_ANDROID_LINKER = "$env:ANDROID_NDK_HOME\toolchains\llvm\prebuilt\windows-x86_64\bin\aarch64-linux-android34-clang.cmd"
$env:CLANG_PATH = "$env:ANDROID_NDK_HOME\toolchains\llvm\prebuilt\windows-x86_64\bin\clang.cmd"

# 以 release 模式编译 ARM64 架构
cargo build --release --target=aarch64-linux-android
```

## 2. 体积优化 (Cargo.toml)

为了将最终二进制文件大小从 ~26MB 减小到 **~7.4MB**，我们修改了 `Cargo.toml` 文件，添加了一个自定义的 `release` 配置文件：

```toml
[profile.release]
strip = true        # 移除调试符号 (显著减小体积)
lto = true          # 链接时优化 (跨 crate 优化)
codegen-units = 1   # 最大化优化 (编译速度变慢)
opt-level = "z"     # 优先优化二进制大小
```

## 3. 故障排除

### 错误："Identifier 'IrohClient' not declared" (标识符未声明)
**原因：** Godot 找不到 GDExtension，因为 `addons` 文件夹不在当前活动的项目目录中。
**解决方案：** 将 `addons` 文件夹从仓库根目录复制到示例项目文件夹 (例如 `examples/chat_app/`)。

### `godot` 0.4.2 编译错误
**原因：** `godot` crate 的 0.4.2 版本对 `String` 到 `GString` 的转换要求更严格。
**解决方案：** 将隐式的 `.into()` 转换更改为使用引用的显式转换：
```rust
// 修改前 (错误)
GString::from(string_variable)
// 修改后 (正确)
GString::from(&string_variable)
```

### 产物位置
编译后的文件生成在：
`target/aarch64-linux-android/release/libgodot_iroh.so`

必须将其复制到插件文件夹以便 Godot 加载：
`addons/godot_iroh/libgodot_iroh_arm64.so`
