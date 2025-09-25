# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

XMRig NVIDIA is a high-performance Monero (XMR) cryptocurrency miner that utilizes NVIDIA GPUs. This is the GPU-accelerated variant of the XMRig mining software, designed specifically for NVIDIA CUDA-compatible graphics cards.

**Current Objective**: Enable and optimize XMRig NVIDIA to run effectively on the NVIDIA Orin Nano platform with its ARM64 architecture and Ampere GPU.

## Architecture

The codebase follows a modular architecture with these key components:

### Core Components
- **App/Controller**: Main application lifecycle and control (`src/App.cpp`, `src/core/Controller.cpp`)
- **Workers**: CUDA mining worker threads (`src/workers/CudaWorker.cpp`, `src/workers/CudaThread.cpp`)
- **NVIDIA Integration**: CUDA kernels and GPU management (`src/nvidia/`)
- **Network**: Mining pool communication and job management (`src/net/`, `src/common/net/`)
- **Configuration**: JSON-based configuration system (`src/core/Config.cpp`, `src/common/config/`)
- **Logging**: Multi-backend logging system (`src/common/log/`)

### CUDA Implementation
- CUDA kernels located in `src/nvidia/cuda_*.cu` files
- CryptoNight algorithm implementations for various GPU architectures
- GPU health monitoring via NVML API (`src/nvidia/NvmlApi.cpp`)
- Support for multiple CUDA compute capabilities (2.0-7.0+)

### Platform Support
- Cross-platform with Windows, Linux, and macOS support
- Platform-specific implementations in `*_win.cpp`, `*_unix.cpp`, `*_mac.cpp` files
- **Target Platform**: NVIDIA Orin Nano (ARM64 architecture with Ampere GPU)

## Build System

### Requirements
- CMake 2.8+
- CUDA Toolkit 8.0+
- C++11 compatible compiler
- libuv library
- OpenSSL (optional, for TLS support)

#### Orin Nano Target Requirements
- JetPack SDK with CUDA support
- ARM64 cross-compilation toolchain (if building on x86_64)
- CUDA compute capability 8.7 (Ampere architecture)
- Power and thermal management considerations

### Build Commands
```bash
# Basic build
mkdir build && cd build
cmake ..
make -j$(nproc)

# With specific CUDA architecture
cmake -DCUDA_ARCH="50;60;70" ..

# Static build
cmake -DBUILD_STATIC=ON ..

# Debug build
cmake -DWITH_DEBUG_LOG=ON ..

# Orin Nano build (ARM64 + Ampere GPU)
cmake -DWITH_HTTPD=OFF -DCUDA_ARCH="87" -DARM_TARGET=8 ..
make -j$(nproc)

# Alternative: suppress deprecated architecture warnings
cmake -DWITH_HTTPD=OFF -DCUDA_ARCH="87" -DARM_TARGET=8 -DCUDA_NVCC_FLAGS="-Wno-deprecated-gpu-targets" ..
```

### Configuration Options
- `WITH_AEON`: CryptoNight-Lite support (default: ON)
- `WITH_SUMO`: CryptoNight-Heavy support (default: ON)
- `WITH_TLS`: OpenSSL/TLS support (default: ON)
- `WITH_HTTPD`: HTTP REST API (default: ON)
- `CUDA_ARCH`: Target GPU architectures (default: 30;50;60;70)
  - For Orin Nano: use `87` (Ampere architecture)
- `ARM_TARGET`: Force ARM target version (8 for ARMv8/AArch64)
- `BUILD_STATIC`: Static binary build (default: OFF)

## Development

### Code Structure
- Headers follow the same directory structure as implementation files
- Interface classes are in `src/*/interfaces/` subdirectories
- Platform-specific code is clearly separated with filename suffixes
- Third-party dependencies are in `src/3rdparty/`

### Key Interfaces
- `IWorker`: Base worker interface
- `IStrategy`: Mining pool strategy interface
- `IConfig`: Configuration interface
- `ILogBackend`: Logging backend interface

### CUDA Development
- CUDA source files use `.cu` extension
- Kernel implementations support multiple GPU architectures
- GPU-specific optimizations in `src/nvidia/cuda_*.hpp` headers
- NVML integration for GPU monitoring and health checks

## Configuration

The miner uses JSON configuration files and supports:
- Multiple mining pools with failover
- GPU-specific tuning parameters
- Algorithm variants (CryptoNight, CryptoNight-Lite, CryptoNight-Heavy)
- API server for monitoring and control
- Donation level configuration (default 5%)

Use [config.xmrig.com/nvidia](https://config.xmrig.com/nvidia) for configuration generation.

## Orin Nano Development Notes

### Key Challenges
- ARM64 architecture compatibility with existing x86_64 codebase
- CUDA compute capability 8.7 support in mining kernels
- Power and thermal constraints of embedded platform
- Cross-compilation and dependency management

### Development Focus Areas
- Verify ARM64 compilation paths in CMake configuration
- Test CUDA kernel compatibility with Ampere architecture (sm_87)
- Optimize power consumption and thermal management
- Validate NVML API functionality on Jetson platform