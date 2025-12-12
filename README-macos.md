# Building GLEW on macOS

This guide summarizes a working macOS build path (tested on Apple Silicon with Xcode command line tools). It covers the GNU Make flow that generates sources and builds the libraries and utilities.

## Prerequisites
- Xcode Command Line Tools (`xcode-select --install`) for `clang`, SDK headers, and `make`.
- `git`, `perl`, and `python3` (preinstalled on macOS; Homebrew versions are fine).
- Optional: Homebrew for convenience (`/opt/homebrew/bin` on Apple Silicon, `/usr/local/bin` on Intel).

## Quick build (recommended)
```sh
# 1) Generate headers/sources from the registries
make -C auto

# 2) Build shared + static libraries and utilities
make -j

# 3) (Optional) Install system-wide
sudo make install   # installs to /usr/local by default

# 4) Clean build outputs
make clean
```
Notes:
- Running `make -C auto` is required when building from a git checkout because `src/glew.c` and headers are generated.
- The default macOS makefile links against the system OpenGL framework; OpenGL is deprecated on macOS but still present.

## Running the utilities from the build tree
If you run `bin/glewinfo` or `bin/visualinfo` without installing, ensure they can find the built dylib:
```sh
DYLD_LIBRARY_PATH="$PWD/lib" bin/glewinfo
```
After `sudo make install`, the tools will load the installed `/usr/local/lib/libGLEW.2.2.0.dylib` and no extra env is needed.

## CMake (alternative)
CMake support is best-effort. A minimal build example:
```sh
mkdir -p build/cmake && cd build/cmake
cmake ../.. -DCMAKE_BUILD_TYPE=Release
cmake --build . --config Release
```
For a framework build, add `-DBUILD_FRAMEWORK=ON` and set `-DCMAKE_INSTALL_PREFIX=/Library/Frameworks`.

## Troubleshooting
- **OpenGL deprecation warnings:** Expected on macOS 10.14+; harmless for building.
- **Loader errors (`Library not loaded: /usr/local/lib/libGLEW.2.2.0.dylib`):**
  - Run the tools with `DYLD_LIBRARY_PATH="$PWD/lib"` (or install system-wide), or
  - Rewrite rpaths/install_name locally: `install_name_tool -change @rpath/libGLEW.2.2.0.dylib @loader_path/../lib/libGLEW.2.2.0.dylib bin/visualinfo`.
- **Missing generated sources:** Always run `make -C auto` before the main build when using a repo checkout.
