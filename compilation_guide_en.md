# Compilation and Troubleshooting Guide - Godot Iroh

This document summarizes the commands used to compile the project for Android, the optimization settings, and the solutions to the problems encountered.

## 1. Android Compilation Command (PowerShell)

To compile the native library for Android (`aarch64-linux-android`), you must configure the NDK environment variables before running `cargo build`.

```powershell
# Configure environment variables for the NDK linker and clang
$env:CARGO_TARGET_AARCH64_LINUX_ANDROID_LINKER = "$env:ANDROID_NDK_HOME\toolchains\llvm\prebuilt\windows-x86_64\bin\aarch64-linux-android34-clang.cmd"
$env:CLANG_PATH = "$env:ANDROID_NDK_HOME\toolchains\llvm\prebuilt\windows-x86_64\bin\clang.cmd"

# Run the compilation in release mode for the ARM64 architecture
cargo build --release --target=aarch64-linux-android
```

## 2. Size Optimization (Cargo.toml)

To reduce the final binary size from ~26MB to **~7.4MB**, we modified the `Cargo.toml` file by adding a custom `release` profile:

```toml
[profile.release]
strip = true        # Removes debug symbols (significantly reduces size)
lto = true          # Link-Time Optimization (optimizes across crates)
codegen-units = 1   # Maximizes optimization (makes compilation slower)
opt-level = "z"     # Optimizes prioritizing binary size
```

## 3. Troubleshooting

### Error: "Identifier 'IrohClient' not declared"
**Cause:** Godot could not find the GDExtension because the `addons` folder was not in the active project directory.
**Solution:** Copy the `addons` folder from the repository root to the example project folder (e.g., `examples/chat_app/`).

### Compilation Error with `godot` 0.4.2
**Cause:** Version 0.4.2 of the `godot` crate is stricter with `String` to `GString` conversions.
**Solution:** Change implicit `.into()` conversions to explicit conversions using references:
```rust
// Before (Error)
GString::from(string_variable)
// After (Correct)
GString::from(&string_variable)
```

### Artifact Location
The compiled file is generated at:
`target/aarch64-linux-android/release/libgodot_iroh.so`

It must be copied to the plugin folder for Godot to load it:
`addons/godot_iroh/libgodot_iroh_arm64.so`
