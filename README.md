# SlicerLookingGlass - Bridge SDK Integration Project

## 📋 Project Overview

This project updates the **SlicerLookingGlass** 3D Slicer extension and its underlying **LookingGlassVTKModule** to support newer Looking Glass holographic displays using the **Bridge SDK 2.6.0** instead of the deprecated HoloPlay Core SDK.

### Goal
Enable support for newer Looking Glass displays:
- **27" Light Field Display** (5K resolution)
- **86" Hololuminescent Display**

While maintaining backward compatibility with existing displays (8K, 4K, Portrait).

---

## 📁 Repository Structure

```
RA/
├── LookingGlassVTKModule/          # VTK rendering module (main SDK integration)
│   ├── bridge_sdk/                 # Bridge SDK 2.6.0 header files
│   │   ├── bridge.h                # Main SDK API
│   │   ├── callbacks.h             # Callback definitions
│   │   └── PixelFormats.h          # Pixel format enums
│   ├── vtkLookingGlassSDKAdapter.h/.cxx    # NEW: Bridge SDK adapter class
│   ├── vtkLookingGlassInterface.h/.cxx     # Device interface (updated for 86")
│   ├── vtkLookingGlassPass.h/.cxx          # Render pass
│   ├── CMakeLists.txt              # Updated for Bridge SDK
│   ├── FindBridgeSDK.cmake         # NEW: CMake module to find Bridge SDK
│   └── ...
├── SlicerLookingGlass/             # 3D Slicer extension wrapper
│   ├── LookingGlass/               # Extension source code
│   ├── CMakeLists.txt              # Updated for VTK 9+
│   ├── FetchVTKRenderingLookingGlass.cmake # Configurable VTK module version
│   └── ...
└── Documentation files (*.md)
```

---

## ✅ Completed Work

### 1. Bridge SDK Integration (LookingGlassVTKModule)

#### New Files Created:

**`vtkLookingGlassSDKAdapter.h/.cxx`** - Bridge SDK wrapper class
- Provides unified C++ interface to Bridge SDK 2.6.0
- Handles SDK initialization/shutdown
- Device enumeration and detection
- Calibration data retrieval
- Cross-platform support (Windows/macOS/Linux)

```cpp
// Usage example
vtkLookingGlassSDKAdapter adapter;
if (adapter.Initialize("SlicerLookingGlass"))
{
    int numDevices = adapter.GetNumDevices();
    for (auto displayIndex : adapter.GetDisplayIndices())
    {
        DeviceInfo info;
        adapter.GetDeviceInfo(displayIndex, info);
        // Use device info...
    }
}
adapter.Shutdown();
```

**`FindBridgeSDK.cmake`** - CMake module for SDK detection
- Searches standard installation paths
- Supports environment variable `BRIDGE_SDK_ROOT`
- Supports local headers in `bridge_sdk/` folder
- Creates proper CMake imported target

**`bridge_sdk/`** - SDK header files
- `bridge.h` - Main API (30+ functions)
- `callbacks.h` - Event callbacks
- `PixelFormats.h` - Pixel format definitions

#### Modified Files:

**`CMakeLists.txt`**
- Removed HoloPlay Core SDK dependency
- Added Bridge SDK as required dependency
- Added `vtkLookingGlassSDKAdapter` to build

**`vtkLookingGlassInterface.cxx`**
- Added device settings for **86" Hololuminescent Display**

### 2. Build System Updates (SlicerLookingGlass)

**`CMakeLists.txt`**
- Updated to support VTK 9 and newer versions (VTK 10+)
- Better version detection and messaging

**`FetchVTKRenderingLookingGlass.cmake`**
- Made VTK module version configurable via `LOOKINGGLASS_VTK_MODULE_GIT_TAG`
- Allows testing with different versions

---

## 🔧 Bridge SDK API Implementation

The `vtkLookingGlassSDKAdapter` class implements the following Bridge SDK functions:

### Initialization
| Function | Description |
|----------|-------------|
| `initialize_bridge()` | Initialize the SDK |
| `uninitialize_bridge()` | Cleanup resources |
| `get_bridge_version()` | Get SDK version |

### Device Enumeration
| Function | Description |
|----------|-------------|
| `get_displays()` | Get list of connected displays |
| `get_device_name_for_display()` | Get device name |
| `get_device_serial_for_display()` | Get serial number |
| `get_device_type_for_display()` | Get hardware type enum |

### Display Properties
| Function | Description |
|----------|-------------|
| `get_dimensions_for_display()` | Screen resolution |
| `get_window_position_for_display()` | Desktop position |
| `get_displayaspect_for_display()` | Aspect ratio |
| `get_viewcone_for_display()` | View cone angle |

### Calibration Parameters
| Function | Description |
|----------|-------------|
| `get_calibration_for_display()` | Full calibration data |
| `get_pitch_for_display()` | Pitch value |
| `get_tilt_for_display()` | Tilt/slope value |
| `get_center_for_display()` | Center offset |
| `get_subp_for_display()` | Subpixel value |
| `get_invview_for_display()` | View inversion flag |
| `get_ri_for_display()` | R index |
| `get_bi_for_display()` | B index |
| `get_fringe_for_display()` | Fringe value |

### Quilt Settings
| Function | Description |
|----------|-------------|
| `get_default_quilt_settings_for_display()` | Recommended quilt dimensions |

---

## 🛠️ Build Requirements

### Software Requirements
- **CMake** 3.15+
- **C++ Compiler** with C++11 support
  - Windows: Visual Studio 2019/2022
  - macOS: Xcode
  - Linux: GCC 7+
- **VTK** 9.0+ (or build with 3D Slicer)
- **Looking Glass Bridge** 2.6.0+ installed

### 3D Slicer Build
To build as part of 3D Slicer:
- **3D Slicer** 5.2.0+ (tested with 5.2.0 and 5.6.2)

---

## 📦 Build Instructions

### Option 1: Build with 3D Slicer Extension

1. Clone this repository
2. Configure 3D Slicer build with extension
3. Set the VTK module path

```bash
# Configure SlicerLookingGlass extension
cmake -S SlicerLookingGlass -B build \
  -DSlicer_DIR=/path/to/Slicer-build/Slicer-build \
  -DLOOKINGGLASS_VTK_MODULE_GIT_TAG=<commit>
```

### Option 2: Standalone VTK Module Build

```bash
cd LookingGlassVTKModule
mkdir build && cd build
cmake .. \
  -DVTK_DIR=/path/to/VTK-build \
  -DBRIDGE_SDK_ROOT=/path/to/bridge/sdk
cmake --build .
```

### Bridge SDK Location

The `FindBridgeSDK.cmake` searches these locations:

**Windows:**
- `C:/Program Files/Looking Glass/Looking Glass Bridge/`
- Environment variable: `BRIDGE_SDK_ROOT`
- Local: `LookingGlassVTKModule/bridge_sdk/`

**macOS:**
- `/Applications/Looking Glass Bridge.app/Contents/`

**Linux:**
- `/usr/local/include/bridge/`
- `/opt/lookingglass/bridge/`

---

## ⏳ Pending Tasks

1. **Build & Compilation Testing** - Verify the code compiles with actual VTK
2. **Hardware Testing** - Test with actual Looking Glass displays
3. **Integration Testing** - Test within 3D Slicer
4. **Performance Optimization** - Optimize for high-resolution displays

---

## 📊 Supported Displays

| Display | Resolution | SDK | Status |
|---------|------------|-----|--------|
| Looking Glass 8K | 8192×8192 quilt | Bridge | ✅ Supported |
| Looking Glass 4K | 4096×4096 quilt | Bridge | ✅ Supported |
| Looking Glass Portrait | 3360×3360 quilt | Bridge | ✅ Supported |
| Looking Glass 27" | 5120×2880 native | Bridge | 🔄 In Progress |
| Looking Glass 86" | 8192×8192 quilt | Bridge | 🔄 In Progress |

---

## 📚 Documentation

- [`FIRST_WEEK_PROGRESS_REPORT.md`](FIRST_WEEK_PROGRESS_REPORT.md) - Detailed progress report
- [`LookingGlassVTKModule/BUILD_INSTRUCTIONS.md`](LookingGlassVTKModule/BUILD_INSTRUCTIONS.md) - Build guide
- [`LookingGlassVTKModule/BRIDGE_SDK_INTEGRATION.md`](LookingGlassVTKModule/BRIDGE_SDK_INTEGRATION.md) - SDK integration details
- [`SlicerLookingGlass/UPDATES.md`](SlicerLookingGlass/UPDATES.md) - Extension updates

---

## 🔗 References

- [3D Slicer](https://www.slicer.org/)
- [Looking Glass Factory](https://lookingglassfactory.com/)
- [Looking Glass Bridge](https://lookingglassfactory.com/software/looking-glass-bridge)
- [Original SlicerLookingGlass](https://github.com/KitwareMedical/SlicerLookingGlass)
- [Original LookingGlassVTKModule](https://github.com/Kitware/LookingGlassVTKModule)

---

## 📝 License

- **SlicerLookingGlass Extension**: Apache 2.0 License
- **LookingGlassVTKModule**: BSD License

---

## 👤 Author

Research Assistant Project - Updating SlicerLookingGlass for newer Looking Glass displays

**Project Start**: February 2026

