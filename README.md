# DirectX 11 Overlay with Python

A guide for creating a DirectX 11 overlay using C++ and interfacing it with Python through a DLL.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Development Environment Setup](#development-environment-setup)
- [Project Setup](#project-setup)
- [Implementation](#implementation)
  - [DirectX 11 Setup](#directx-11-setup)
  - [DLL Export Functions](#dll-export-functions)
  - [Python Integration](#python-integration)
- [Testing](#testing)
- [Troubleshooting](#troubleshooting)

## Prerequisites
- Visual Studio (Community Edition or higher)
- Windows SDK
- Python 3.x
- Basic knowledge of C++ and Python

## Development Environment Setup

### Installing Visual Studio
1. Download Visual Studio Community from the [Visual Studio Downloads page](https://visualstudio.microsoft.com/downloads/)
2. During installation, select "Desktop Development with C++" workload
3. In the Individual Components section, ensure "Windows 10/11 SDK" is selected

## Project Setup

### Creating the DLL Project
1. Open Visual Studio
2. Go to `File > New > Project`
3. Select "Dynamic Link Library (DLL)" under C++ projects
4. Name your project (e.g., `DirectXOverlay`)
5. Click Create

### Configuring DirectX 11
1. Right-click project in Solution Explorer → Properties
2. Navigate to C/C++ → General
3. Add to Additional Include Directories:
   ```
   $(WindowsSDK_IncludePath)
   ```
4. Navigate to Linker → Input
5. Add to Additional Dependencies:
   ```
   d3d11.lib
   dxgi.lib
   ```

## Implementation

### DirectX 11 Setup

First, include the necessary headers in your main `.cpp` file:

```cpp
#include <Windows.h>
#include <d3d11.h>
```

Declare global DirectX variables:

```cpp
IDXGISwapChain* swapChain = nullptr;
ID3D11Device* device = nullptr;
ID3D11DeviceContext* context = nullptr;
ID3D11RenderTargetView* renderTargetView = nullptr;
```

### DLL Export Functions

#### Initialize Overlay
```cpp
extern "C" __declspec(dllexport) bool initialize_overlay(HWND hwnd) {
    DXGI_SWAP_CHAIN_DESC scd = {};
    scd.BufferCount = 1;
    scd.BufferDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;
    scd.BufferUsage = DXGI_USAGE_RENDER_TARGET_OUTPUT;
    scd.OutputWindow = hwnd;
    scd.SampleDesc.Count = 1;
    scd.Windowed = TRUE;

    HRESULT hr = D3D11CreateDeviceAndSwapChain(
        nullptr, 
        D3D_DRIVER_TYPE_HARDWARE, 
        nullptr, 
        0,
        nullptr, 
        0, 
        D3D11_SDK_VERSION, 
        &scd, 
        &swapChain, 
        &device, 
        nullptr, 
        &context
    );

    if (FAILED(hr)) {
        return false;
    }

    ID3D11Texture2D* backBuffer = nullptr;
    swapChain->GetBuffer(0, __uuidof(ID3D11Texture2D), (void**)&backBuffer);
    device->CreateRenderTargetView(backBuffer, nullptr, &renderTargetView);
    backBuffer->Release();

    return true;
}
```

#### Render Frame
```cpp
extern "C" __declspec(dllexport) void render_frame(float r, float g, float b, float a) {
    const float color[4] = { r, g, b, a };
    context->ClearRenderTargetView(renderTargetView, color);
    swapChain->Present(1, 0);
}
```

#### Shutdown Overlay
```cpp
extern "C" __declspec(dllexport) void shutdown_overlay() {
    if (renderTargetView) renderTargetView->Release();
    if (swapChain) swapChain->Release();
    if (context) context->Release();
    if (device) device->Release();
}
```

### Python Integration

```python
import ctypes
import time

# Load the DLL
dx_overlay = ctypes.CDLL('./DirectXOverlay.dll')

# Define function arguments and return types
dx_overlay.initialize_overlay.argtypes = [ctypes.c_void_p]
dx_overlay.initialize_overlay.restype = ctypes.c_bool
dx_overlay.render_frame.argtypes = [ctypes.c_float, ctypes.c_float, ctypes.c_float, ctypes.c_float]
dx_overlay.shutdown_overlay.restype = None

# Initialize and run the overlay
hwnd = 0  # Use a valid HWND from a created window in a real scenario

if dx_overlay.initialize_overlay(hwnd):
    print("Overlay initialized.")

    try:
        # Render loop
        while True:
            # Render with a cyan color
            dx_overlay.render_frame(0.0, 0.5, 0.5, 1.0)
            time.sleep(1 / 60)  # Approx 60 FPS

    except KeyboardInterrupt:
        print("Exiting rendering loop.")
    finally:
        dx_overlay.shutdown_overlay()
else:
    print("Failed to initialize overlay.")
```

## Testing
1. Build the DLL project in Visual Studio
2. Locate the compiled `.dll` file in the Debug or Release folder
3. Run the Python script in the same directory as the DLL
4. Verify that the overlay initializes and renders correctly

## Troubleshooting
- Ensure all DirectX dependencies are properly linked
- Check that the DLL is in the same directory as your Python script
- Verify that you're using a valid window handle (HWND)
- Monitor the console for initialization errors
- If the overlay doesn't appear, check your window Z-order and transparency settings

## Contributing
Feel free to submit issues and enhancement requests!

## License
[Your chosen license]

---
Created with ❤️ using DirectX 11 and Python
