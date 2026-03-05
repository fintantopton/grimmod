# GrimMod: Mod Loader for Grim Fandango Remastered

Download at [https://hexagon.codes/grimhd](https://hexagon.codes/grimhd)

## Features

* **Mods**
  - Allows for the creation of asset mods that can swap out any file usually loaded from the game's .LAB datapacks.
* **High-Quality Assets**
  - Upgrades the renderer to support high-quality assets with any resolution and 32-bit color. Without this, assets are max 640x480 with dithered 24-bit color.
* **Forced VSync**
  - Previously, the remaster didn't use vsync while in-game and produced frames as fast as it could (sometimes causing coil whine).
* **High-DPI Fix** (Windows/macOS)
  - On high-dpi systems with UI scaling above 100%, the remaster renders at a lower resolution and lets the system scale it up. This isn't that noticeable with 640x480 backgrounds but it stops high-quality assets from displaying at their full resolution. GrimMod forces the game to always render at the window's actual resolution.
* **Quick Renderer Toggle**
  - In the remaster, toggling the renderer between Original/Remastered is a smooth transition which makes changes less noticeable. GrimMod makes the toggle instant, highlighting the differences between the renderers.

## Installation

### Windows

1. Place `glu32.dll` in the root of the Grim Fandango Remastered directory where `GrimFandango.exe` is located.
2. (Optional) Put any mods in a `Mods` folder in the root directory.
3. Enjoy the ride!

Note: For GOG on Windows, when launching with the "Launch Grim Fandango Remastered" shortcut (like GOG Galaxy does), it's necessary to locate the shortcut in the game folder and set "Properties -> Compatibility -> Run this program as an administrator".

Note: For the Steam Deck, first set the game to run the Windows version by using Proton in the compatibility options.

### Linux (GOG)

1. Place `libgrimmod.so` in the game's `bin` directory alongside the `GrimFandango` executable (typically `<install-path>/game/bin/`).
2. Launch the game with `LD_PRELOAD` pointing to the library:
   ```bash
   cd <install-path>/game/bin
   LD_PRELOAD=./libgrimmod.so ./GrimFandango
   ```
   Alternatively, edit the GOG launch script (`start.sh`) and add the following line before the game is executed:
   ```bash
   export LD_PRELOAD="$bin_path/libgrimmod.so"
   ```
3. (Optional) Put any mods in a `Mods` folder inside the `bin` directory (i.e. `<install-path>/game/bin/Mods/`).
4. (Optional) Create a `grimmod.toml` in the `bin` directory to configure options (see [Config](#config)).

Note: When running the native Linux build under **box64** (e.g. on non-x86 hardware), use `BOX64_LD_PRELOAD` instead of `LD_PRELOAD`:
```bash
export BOX64_LD_PRELOAD="$bin_path/libgrimmod.so"
```

### macOS

The macOS version works with the App Store/GOG build of Grim Fandango Remastered. It runs under Rosetta 2 on Apple Silicon Macs.

**Step 1: Place the library**

Copy `libgrimmod.dylib` into the app bundle's `MacOS` directory:
```bash
cp libgrimmod.dylib "/Applications/Grim Fandango Remastered.app/Contents/MacOS/"
```

**Step 2: Re-sign the game binary**

The game binary ships with a hardened runtime signature that blocks library injection. You need to re-sign it with entitlements that allow `DYLD_INSERT_LIBRARIES`.

Create an `entitlements.plist` file:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>com.apple.security.cs.allow-dyld-environment-variables</key>
    <true/>
    <key>com.apple.security.cs.disable-library-validation</key>
    <true/>
</dict>
</plist>
```

Back up and re-sign the binary:
```bash
APP="/Applications/Grim Fandango Remastered.app/Contents/MacOS"
cp "$APP/GrimFandango" "$APP/GrimFandango.original"
codesign --force --sign - --entitlements entitlements.plist --deep "$APP/GrimFandango"
```

**Step 3: Install the launcher wrapper**

Rename the game binary and create a wrapper script so grimmod loads automatically when you launch the app:
```bash
APP="/Applications/Grim Fandango Remastered.app/Contents/MacOS"
mv "$APP/GrimFandango" "$APP/GrimFandango.real"
cat > "$APP/GrimFandango" << 'EOF'
#!/bin/bash
DIR="$(dirname "$0")"
export DYLD_INSERT_LIBRARIES="$DIR/libgrimmod.dylib"
exec "$DIR/GrimFandango.real" "$@"
EOF
chmod +x "$APP/GrimFandango"
```

The game will now load grimmod automatically when launched from Finder, Dock, Spotlight, or the command line.

**Step 4: Install mods (optional)**

Place mod folders in the app bundle's `Resources` directory:
```bash
cp -r <mod-folder> "/Applications/Grim Fandango Remastered.app/Contents/Resources/Mods/"
```

**Step 5: Configure (optional)**

Create a `grimmod.toml` in the `Resources` directory to configure options (see [Config](#config)):
```bash
cp grimmod.toml "/Applications/Grim Fandango Remastered.app/Contents/Resources/"
```

**Uninstalling**: To restore the original game, remove the wrapper and rename the binary back:
```bash
APP="/Applications/Grim Fandango Remastered.app/Contents/MacOS"
rm "$APP/GrimFandango" "$APP/libgrimmod.dylib"
mv "$APP/GrimFandango.real" "$APP/GrimFandango"
```

## Changelog
  ### 3.0.0
  - Added macOS support (App Store/GOG version via `DYLD_INSERT_LIBRARIES`, works on Apple Silicon via Rosetta 2).
  - Fixed 64-bit struct layouts for `RenderPassEntity`, `Draw`, `Surface`, `ImageContainer`, `Image`, and `ImageAttributes`.
  - Fixed `draw_indexed_primitives` parameter truncation on 64-bit (index buffer pointer).
  - Fixed `copy_image` parameter truncation on 64-bit (`LECRECT*` pointer).
  ### 2.0.0
  - Added Linux support (native GOG Linux version via `LD_PRELOAD`).
  - Fixed CString memory leak in modded file open path.
  - Fixed struct layout for `RenderPassEntity` and `Draw` (renderer correctness).
  - Fixed off-by-one in `Vector<T>::len()`.
  - PNG loading on Linux uses lenient checksum validation (required for GrimHD mod assets).
  ### 1.1.0
  - Added GOG support (see installation note).
  - Added Steam Deck support (see installation note).
  - Improved initialization stability with better logging for errors.
  ### 1.0.0
  - Initial release.

## Limitations

* Doesn't allow for upscaling videos, yet. This includes full cutscenes and scenes that use video as part of the background/foreground (however most animations are simply a series of images, which can be upscaled).
* Doesn't attempt to make the game 16:9. Any non-4:3 assets will still look stretched.
* A full playthrough with grimmod has been completed but as new software, bugs and crashes are to be expected. Save regularly (but autosave is a potential future feature!).

## Config

Create a `grimmod.toml` file beside the game binary (`glu32.dll` on Windows, `GrimFandango` on Linux, or in `Contents/Resources/` on macOS) to tweak options:

| Setting                               | Default          | Effect |
| ------------------------------------- | ---------------- | ------ |
| `mods = true/false`                   | true             | Enable/disable the loading of mods |
| `renderer.hq_assets = true/false`     | true             | Enable/disable hooking the renderer to load modern image formats (PNG/VP9 MKV) from mods |
| `renderer.quick_toggle = true/false`  | true             | Enable for instant toggling between the Original/Remastered renderers, disable to restore the smooth transition |
| `renderer.video_cutouts = true/false` | true             | Some scenes use videos, which are not yet upscalable with GrimMod, as the entire background image. This option allows GrimMod to manually carve out static chunks of the video, exposing the background underneath. As a somewhat hacky solution it has been given its own toggle if issues pop up. |
| `display.vsync = true/false`          | true             | Enable/disable forced VSync |
| `display.hdpi_fix = true/false`       | true (Win/macOS) / false (Linux) | GrimMod rewrites some of the window handling to always render at native resolution. Not applicable on Linux. |
| `logging.enabled = true/false`        | true             | Enable/disable creation of and writing to `grimmod.log` with simple logging info, mostly for the purposes of a health check. |
| `logging.debug = true/false`          | false            | Enable/disable debug logging. This outputs a lot of information per frame, useless outside of debugging/development. |

## Building

The project requires Rust Nightly (uses `#![feature(fn_traits, tuple_trait, unboxed_closures)]`) and libvpx.

### Windows

```bash
cargo +nightly build --release --target i686-pc-windows-msvc
```

The output is `target/i686-pc-windows-msvc/release/grimmod.dll`, which should be renamed to `glu32.dll` for deployment.

### Linux

The Linux build targets 32-bit x86 (`i686-unknown-linux-gnu`) and requires `gcc-multilib` and 32-bit `libvpx` development headers. Using Docker:

```bash
docker run --rm --platform linux/amd64 -v "$(pwd)":/src -w /src rust:latest bash -c "
  dpkg --add-architecture i386 &&
  apt-get update &&
  apt-get install -y gcc-multilib libc6-dev-i386 libvpx-dev:i386 pkg-config:i386 &&
  rustup toolchain install nightly &&
  rustup +nightly target add i686-unknown-linux-gnu &&
  export PKG_CONFIG_ALLOW_CROSS=1 &&
  export PKG_CONFIG_PATH=/usr/lib/i386-linux-gnu/pkgconfig &&
  cargo +nightly build --release --target i686-unknown-linux-gnu
"
```

The output is `target/i686-unknown-linux-gnu/release/libgrimmod.so`.

### macOS

The macOS build targets 64-bit x86_64 (`x86_64-apple-darwin`) and requires `libvpx` built for x86_64. On Apple Silicon, Rust cross-compiles to x86_64 and the game runs under Rosetta 2.

Build libvpx for x86_64 (if not already installed):
```bash
git clone https://chromium.googlesource.com/webm/libvpx
cd libvpx
git checkout v1.16.0
mkdir build-x86_64 && cd build-x86_64
CROSS=x86_64-apple-darwin ../configure --target=x86_64-darwin20-gcc \
  --disable-examples --disable-tools --disable-unit-tests \
  --enable-static --disable-shared --prefix=/tmp/libvpx-x86_64
make -j$(sysctl -n hw.ncpu) && make install
```

Build grimmod:
```bash
rustup +nightly target add x86_64-apple-darwin
PKG_CONFIG_ALLOW_CROSS=1 PKG_CONFIG_PATH=/tmp/libvpx-x86_64/lib/pkgconfig \
  cargo +nightly build --release --target x86_64-apple-darwin
```

The output is `target/x86_64-apple-darwin/release/libgrimmod.dylib`.

## Architecture

GrimMod works by hooking game functions at runtime to intercept asset loading and rendering calls.

- **Windows**: Injected as a DLL proxy (`glu32.dll`). Uses `retour` for inline function hooking, IAT overwriting for indirect hooks, and `lightningscanner` for byte-pattern scanning to locate game functions.
- **Linux**: Injected via `LD_PRELOAD`. Uses inline x86 prologue patching with mmap'd trampolines for direct hooks, GOT overwriting for indirect hooks, and ELF symtab parsing (via `goblin`) to resolve game symbols.
- **macOS**: Injected via `DYLD_INSERT_LIBRARIES`. Uses inline x86-64 prologue patching with MAP_JIT trampolines for direct hooks, Mach-O lazy/non-lazy symbol pointer overwriting for indirect hooks, and Mach-O nlist/LC_SYMTAB parsing (via `goblin`) with `dlsym` for symbol resolution. Requires re-signing the game binary with entitlements to allow library injection. The macOS binary is 64-bit x86_64 (vs 32-bit i386 on Linux/Windows), requiring adjusted struct layouts for all pointer-containing engine structs.
