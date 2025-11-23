# Guía de Compilación y Solución de Problemas - Godot Iroh

Este documento resume los comandos utilizados para compilar el proyecto para Android, las configuraciones de optimización y las soluciones a los problemas encontrados.

## 1. Comando de Compilación para Android (PowerShell)

Para compilar la librería nativa para Android (`aarch64-linux-android`), se deben configurar las variables de entorno del NDK antes de ejecutar `cargo build`.

```powershell
# Configurar variables de entorno para el linker y clang del NDK
$env:CARGO_TARGET_AARCH64_LINUX_ANDROID_LINKER = "$env:ANDROID_NDK_HOME\toolchains\llvm\prebuilt\windows-x86_64\bin\aarch64-linux-android34-clang.cmd"
$env:CLANG_PATH = "$env:ANDROID_NDK_HOME\toolchains\llvm\prebuilt\windows-x86_64\bin\clang.cmd"

# Ejecutar la compilación en modo release para la arquitectura ARM64
cargo build --release --target=aarch64-linux-android
```

## 2. Optimización de Tamaño (Cargo.toml)

Para reducir el tamaño del binario final de ~26MB a **~7.4MB**, modificamos el archivo `Cargo.toml` agregando un perfil de `release` personalizado:

```toml
[profile.release]
strip = true        # Elimina símbolos de depuración (reduce mucho el tamaño)
lto = true          # Link-Time Optimization (optimiza entre crates)
codegen-units = 1   # Maximiza la optimización (hace la compilación más lenta)
opt-level = "z"     # Optimiza priorizando el tamaño del binario
```

## 3. Solución de Problemas Encontrados

### Error: "Identifier 'IrohClient' not declared"
**Causa:** Godot no podía encontrar la extensión GDExtension porque la carpeta `addons` no estaba en el directorio del proyecto activo.
**Solución:** Copiar la carpeta `addons` desde la raíz del repositorio a la carpeta del proyecto de ejemplo (e.g., `examples/chat_app/`).

### Error de Compilación con `godot` 0.4.2
**Causa:** La versión 0.4.2 de la crate `godot` es más estricta con las conversiones de `String` a `GString`.
**Solución:** Cambiar las conversiones implícitas `.into()` por conversiones explícitas usando referencias:
```rust
// Antes (Error)
GString::from(string_variable)
// Después (Correcto)
GString::from(&string_variable)
```

### Ubicación del Artefacto
El archivo compilado se genera en:
`target/aarch64-linux-android/release/libgodot_iroh.so`

Debe copiarse a la carpeta del plugin para que Godot lo cargue:
`addons/godot_iroh/libgodot_iroh_arm64.so`
