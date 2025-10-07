# Building FFmpeg on Windows with MSVC and VS Code Build Tools

This guide provides comprehensive instructions for building FFmpeg on Windows using Microsoft Visual C++ (MSVC) compiler and Visual Studio Code build tools. The purpose is to build a complete FFmpeg binary executable.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Required Downloads and Installations](#required-downloads-and-installations)
- [Environment Setup](#environment-setup)
- [Building FFmpeg](#building-ffmpeg)
- [Verification](#verification)
- [Troubleshooting](#troubleshooting)
- [Optional Dependencies](#optional-dependencies)

---

## Prerequisites

### System Requirements
- **Operating System**: Windows 10 or Windows 11 (64-bit recommended)
- **Disk Space**: At least 5 GB free space
- **RAM**: Minimum 4 GB (8 GB or more recommended)
- **Administrator Access**: Required for installing software

---

## Required Downloads and Installations

### 1. Visual Studio Build Tools (MSVC Compiler)

FFmpeg can be built with MSVC 2013 or later. For modern development, we recommend Visual Studio 2019 or 2022.

#### Option A: Visual Studio Community (Full IDE)
- **Download**: [Visual Studio Community](https://visualstudio.microsoft.com/downloads/)
- **Installation**:
  1. Run the installer
  2. Select **"Desktop development with C++"** workload
  3. Ensure the following components are selected:
     - MSVC v142/v143 - VS 2019/2022 C++ x64/x86 build tools
     - Windows 10/11 SDK
     - C++ CMake tools for Windows (optional but useful)
     - C++ Clang tools for Windows (optional)
  4. Complete the installation (approximately 5-10 GB)

#### Option B: Build Tools for Visual Studio (Lightweight)
- **Download**: [Build Tools for Visual Studio](https://visualstudio.microsoft.com/downloads/#build-tools-for-visual-studio-2022)
- **Installation**:
  1. Run the Build Tools installer
  2. Select **"Desktop development with C++"** workload
  3. Install the same components as Option A
  4. This is a lighter installation without the full IDE (~3-5 GB)

### 2. MSYS2 (Unix-like Environment for Windows)

MSYS2 provides the necessary Unix tools (bash, make, configure scripts) required to build FFmpeg.

- **Download**: [MSYS2](https://www.msys2.org/)
- **Installation**:
  1. Download the installer (msys2-x86_64-*.exe)
  2. Run the installer and follow the prompts
  3. Default installation path: `C:\msys64`
  4. After installation, MSYS2 will open a terminal window
  5. Update the package database:
     ```bash
     pacman -Syu
     ```
  6. Close the terminal when prompted, then reopen MSYS2 and run:
     ```bash
     pacman -Su
     ```

### 3. Install MSYS2 Packages

Open the **MSYS2 MSYS** terminal (not MinGW64 or MinGW32) and install required packages:

```bash
# Essential build tools
pacman -S make pkgconf diffutils

# Optional but recommended
pacman -S git perl python
```

### 4. NASM Assembler

NASM is required for optimized assembly code in FFmpeg.

#### Option A: Via MSYS2 (Recommended)
```bash
pacman -S nasm
```

#### Option B: Manual Installation
- **Download**: [NASM](https://www.nasm.us/pub/nasm/releasebuilds/)
- Download the latest Windows 64-bit installer (e.g., `nasm-*.win64.exe`)
- Install to a simple path like `C:\nasm`
- Add `C:\nasm` to your Windows PATH environment variable

### 5. Visual Studio Code (Optional but Recommended)

- **Download**: [Visual Studio Code](https://code.visualstudio.com/)
- **Installation**: Standard installation
- **Recommended Extensions**:
  - C/C++ Extension Pack (Microsoft)
  - Makefile Tools
  - Terminal

---

## Environment Setup

### Step 1: Locate Visual Studio Tools

Find your Visual Studio installation path. Common locations:
- Visual Studio 2022: `C:\Program Files\Microsoft Visual Studio\2022\Community`
- Visual Studio 2019: `C:\Program Files (x86)\Microsoft Visual Studio\2019\Community`
- Build Tools: `C:\Program Files (x86)\Microsoft Visual Studio\2019\BuildTools`

### Step 2: Create a Build Script

Create a batch file `build_ffmpeg_env.bat` in a convenient location:

```batch
@echo off
REM Build FFmpeg Environment Setup Script for MSVC

REM Set Visual Studio version and path
REM Adjust this path to match your Visual Studio installation
SET "VS_PATH=C:\Program Files\Microsoft Visual Studio\2022\Community"

REM Initialize Visual Studio environment
call "%VS_PATH%\VC\Auxiliary\Build\vcvars64.bat"

REM Set MSYS2 path
SET "MSYS2_PATH=C:\msys64"

REM Add NASM to PATH (if manually installed)
REM SET "PATH=%PATH%;C:\nasm"

REM Launch MSYS2 shell with MSVC environment
%MSYS2_PATH%\msys2_shell.cmd -msys -defterm -no-start -here
```

**Important Notes**:
- Adjust `VS_PATH` to match your Visual Studio installation
- Use `vcvars64.bat` for 64-bit builds or `vcvars32.bat` for 32-bit builds
- Adjust `MSYS2_PATH` if you installed MSYS2 in a different location

### Step 3: Verify Environment

1. Run your `build_ffmpeg_env.bat` script
2. In the MSYS2 terminal that opens, verify the tools:

```bash
# Check MSVC compiler
which cl.exe

# Check NASM
which nasm

# Check make
which make

# Verify MSVC environment variables
echo $LIB
echo $INCLUDE
```

All commands should return valid paths. If not, review your setup.

---

## Building FFmpeg

### Step 1: Get FFmpeg Source Code

If you haven't already cloned the repository:

```bash
# Clone FFmpeg repository
git clone https://github.com/nicholas-g-lipari-phd/Test-Building-FFmpeg-with-Copilot.git
cd Test-Building-FFmpeg-with-Copilot
```

Or if you already have it:

```bash
cd /path/to/Test-Building-FFmpeg-with-Copilot
```

### Step 2: Configure FFmpeg for MSVC

Run the configure script with MSVC toolchain:

```bash
# Basic configuration for MSVC
./configure --toolchain=msvc

# Recommended configuration with common options
./configure --toolchain=msvc \
    --enable-gpl \
    --enable-version3 \
    --disable-debug \
    --disable-shared \
    --enable-static \
    --prefix=/usr/local

# For shared libraries (DLLs) instead of static
./configure --toolchain=msvc \
    --enable-shared \
    --disable-static \
    --prefix=/usr/local
```

**Important Configuration Notes**:
- `--toolchain=msvc`: Use Microsoft Visual C++ compiler
- `--enable-static` / `--enable-shared`: Choose library type
  - **Cannot use both static and shared simultaneously with MSVC**
  - Static builds create `.lib` files
  - Shared builds create `.dll` and `.lib` files
- `--prefix`: Installation directory (in MSYS2 Unix path format)
- `--enable-gpl`: Enables GPL code (makes binaries GPL-licensed)
- `--disable-debug`: Creates optimized release build

### Step 3: Review Configuration

After running configure, check the output:

```bash
# View the configuration log
cat ffbuild/config.log | tail -100

# Check what was enabled/disabled
./configure --list-decoders
./configure --list-encoders
```

### Step 4: Build FFmpeg

```bash
# Build with all available CPU cores
make -j$(nproc)

# Or build with specific number of jobs
make -j4

# For faster subsequent builds
make -r -j$(nproc)
```

**Build Time**: Expect 10-30 minutes depending on your system and configuration.

### Step 5: Install FFmpeg

```bash
# Install to the prefix directory specified in configure
make install
```

This will install:
- **Binaries**: `ffmpeg.exe`, `ffplay.exe`, `ffprobe.exe`
- **Libraries**: `.lib` files (and `.dll` if shared)
- **Headers**: Include files for development
- **Documentation**: Man pages and documentation

### Step 6: Locate Built Binaries

After building, find your executables:

```bash
# In the build directory
ls -lh ffmpeg.exe ffprobe.exe ffplay.exe

# In the installation directory
ls -lh /usr/local/bin/
```

To use from Windows, the MSYS2 path `/usr/local/bin` typically maps to:
- `C:\msys64\usr\local\bin`

---

## Verification

### Test the Built FFmpeg

```bash
# Check version
./ffmpeg.exe -version

# Check build configuration
./ffmpeg.exe -buildconf

# Simple test - convert a file
./ffmpeg.exe -i input.mp4 -c:v libx264 -c:a aac output.mp4

# Get codec information
./ffmpeg.exe -codecs | head -20
```

### Test from Windows Command Prompt

1. Open Windows Command Prompt (not MSYS2)
2. Navigate to the binary location:
   ```cmd
   cd C:\msys64\usr\local\bin
   ffmpeg.exe -version
   ```

### Add to Windows PATH (Optional)

To use FFmpeg from anywhere in Windows:

1. Copy the executables to a permanent location (e.g., `C:\ffmpeg\bin`)
2. Add that path to Windows PATH environment variable:
   - Right-click "This PC" → Properties
   - Advanced system settings → Environment Variables
   - Edit "Path" in System Variables
   - Add new entry: `C:\ffmpeg\bin`
   - Click OK and restart any open terminals

---

## Troubleshooting

### Common Issues and Solutions

#### Issue: "cl.exe not found"
**Solution**: 
- Ensure you're running MSYS2 from the batch script that calls `vcvars64.bat`
- Verify Visual Studio is properly installed
- Check that `which cl.exe` returns a valid path

#### Issue: "nasm not found"
**Solution**:
- Install NASM via MSYS2: `pacman -S nasm`
- Or add NASM to PATH if manually installed
- Verify with: `which nasm`

#### Issue: "link.exe: command not found" or wrong linker used
**Solution**:
- MSYS2 may have its own `link.exe` that conflicts
- Ensure MSVC paths come first in PATH
- The batch script should properly initialize MSVC environment

#### Issue: Configuration fails with "C compiler test failed"
**Solution**:
- Run configure with `--verbose` to see detailed error
- Check that LIB and INCLUDE environment variables are set
- Verify Visual Studio environment is properly initialized

#### Issue: "Cannot build both shared and static libraries"
**Solution**:
- This is a MSVC limitation
- Choose either `--enable-shared` OR `--enable-static`
- Build twice if you need both: once for static, once for shared

#### Issue: Build fails with undefined symbols or linking errors
**Solution**:
- Ensure all dependencies are properly installed
- Check configure output for disabled features
- Try a minimal configuration first: `./configure --toolchain=msvc --disable-autodetect`

#### Issue: "make: *** No rule to make target"
**Solution**:
- Clean and reconfigure: `make clean && ./configure --toolchain=msvc`
- Ensure GNU Make is being used: `which make`

#### Issue: Runtime error - missing DLL
**Solution** (for shared builds):
- Copy all `.dll` files alongside the executable
- Or add the DLL directory to Windows PATH

---

## Optional Dependencies

### Adding External Libraries

FFmpeg supports many external libraries for additional codecs and formats. Here are common ones:

#### libx264 (H.264 encoder)
```bash
# In MSYS2 MSYS terminal
pacman -S mingw-w64-x86_64-x264

# Then configure FFmpeg with:
./configure --toolchain=msvc --enable-libx264 --enable-gpl \
    --extra-cflags="-I/mingw64/include" \
    --extra-ldflags="-L/mingw64/lib"
```

#### libx265 (H.265/HEVC encoder)
```bash
pacman -S mingw-w64-x86_64-x265
# Add --enable-libx265 to configure
```

#### libvpx (VP8/VP9 codec)
```bash
pacman -S mingw-w64-x86_64-libvpx
# Add --enable-libvpx to configure
```

#### zlib (Compression)
```bash
pacman -S zlib-devel
# Add --enable-zlib to configure
```

**Note**: When mixing MSVC with MinGW libraries, you may encounter compatibility issues. For production builds, it's recommended to either:
1. Build all dependencies with MSVC
2. Use a pure MinGW-w64 toolchain instead
3. Use pre-built MSVC-compatible libraries

### Building with zlib (MSVC-compatible)

For MSVC-compatible zlib:

1. Download zlib source from [zlib.net](http://zlib.net/)
2. Extract to a directory (e.g., `C:\zlib`)
3. Build with MSVC:
   ```cmd
   REM In Visual Studio Developer Command Prompt
   cd C:\zlib
   nmake -f win32\Makefile.msc
   ```
4. Edit `zconf.h` to remove `#include <unistd.h>` (around line 425)
5. Copy files to a location MSVC can find:
   - `zlib.lib` → include in LIB path
   - `zlib.h`, `zconf.h` → include in INCLUDE path
6. Configure FFmpeg:
   ```bash
   ./configure --toolchain=msvc --enable-zlib \
       --extra-cflags="-IC:/zlib" \
       --extra-ldflags="-LIBPATH:C:/zlib"
   ```

---

## Advanced Configuration Options

### Optimization Flags

```bash
# Optimize for size
./configure --toolchain=msvc --enable-small

# Optimize for specific CPU
./configure --toolchain=msvc --cpu=native

# Disable runtime CPU detection (smaller binary)
./configure --toolchain=msvc --disable-runtime-cpudetect
```

### Minimal Build

```bash
# Minimal build - only essential codecs
./configure --toolchain=msvc \
    --disable-autodetect \
    --disable-everything \
    --enable-decoder=h264 \
    --enable-decoder=aac \
    --enable-encoder=libx264 \
    --enable-encoder=aac \
    --enable-demuxer=mov \
    --enable-muxer=mp4 \
    --enable-protocol=file
```

### Full-Featured Build

```bash
# Build with many features enabled
./configure --toolchain=msvc \
    --enable-gpl \
    --enable-version3 \
    --enable-nonfree \
    --enable-runtime-cpudetect \
    --disable-debug \
    --enable-avfilter \
    --enable-avformat \
    --enable-avcodec \
    --enable-swscale \
    --enable-swresample
```

---

## Building for 32-bit Windows

To build for 32-bit (x86) instead of 64-bit:

1. Modify the batch script to use `vcvars32.bat`:
   ```batch
   call "%VS_PATH%\VC\Auxiliary\Build\vcvars32.bat"
   ```

2. Configure with 32-bit target:
   ```bash
   ./configure --toolchain=msvc --arch=x86
   ```

3. Build as normal:
   ```bash
   make -j$(nproc)
   ```

---

## Additional Resources

### Official Documentation
- [FFmpeg Documentation](https://ffmpeg.org/documentation.html)
- [FFmpeg Wiki - Compilation Guide](https://trac.ffmpeg.org/wiki/CompilationGuide)
- See `doc/platform.texi` in this repository for detailed platform-specific information

### Community Resources
- [FFmpeg Users Mailing List](https://ffmpeg.org/contact.html#MailingLists)
- [FFmpeg Bug Tracker](https://trac.ffmpeg.org/)

### Related Files in This Repository
- `INSTALL.md` - General installation instructions
- `doc/platform.texi` - Platform-specific build documentation
- `doc/build_system.txt` - Build system documentation

---

## Summary

You now have a complete guide to build FFmpeg on Windows with MSVC. Key steps:

1. ✅ Install Visual Studio Build Tools or Visual Studio Community
2. ✅ Install MSYS2 and required packages
3. ✅ Install NASM assembler
4. ✅ Set up build environment with batch script
5. ✅ Configure FFmpeg with `--toolchain=msvc`
6. ✅ Build with `make`
7. ✅ Install with `make install`
8. ✅ Test the built binaries

For questions or issues, refer to the Troubleshooting section or consult the official FFmpeg documentation.

**Happy Building!** 🎬🎥🎞️
