# C# DirectX 11 Overlay Tutorial

This guide walks through building a DirectX 11 overlay DLL in C# for creating graphical overlays. We'll use **SharpDX** (DirectX C# wrapper) and configure a transparent Windows Form for display.

## Prerequisites

1. **Visual Studio** (for creating a C# Class Library)
2. **SharpDX NuGet packages** (DirectX API access in C#)
3. **Python** (if you plan to call the DLL from Python)

## Setup Guide

### Step 1: Create a New Class Library Project

1. Open **Visual Studio**
2. Create a new **Class Library** project
   - Select `.NET Framework` for broader compatibility
   - Name it something like `OverlayLibrary`

### Step 2: Install SharpDX Packages

To use DirectX in C#, install the following **SharpDX** packages:

1. Open the **Package Manager Console** in Visual Studio (`Tools` > `NuGet Package Manager` > `Package Manager Console`)
2. Run these commands:

```powershell
Install-Package SharpDX
Install-Package SharpDX.Direct3D11
Install-Package SharpDX.Mathematics
```

### Step 3: Create an Overlay Window

Now we'll set up a transparent window as the overlay:

1. Right-click the project, select Add > Windows Form, and name it `OverlayForm`
2. Open OverlayForm.cs and set the following properties in the constructor:

```csharp
public OverlayForm()
{
    this.FormBorderStyle = FormBorderStyle.None;   // No border
    this.TopMost = true;                           // Always on top
    this.Opacity = 0.75;                           // Semi-transparent
    this.BackColor = Color.Black;                  // Background color
    this.TransparencyKey = Color.Black;            // Make background transparent
}
```

### Step 4: Initialize DirectX 11 in C#

Add DirectX initialization code to render content to the overlay:

1. In your class library, create a new class named `OverlayRenderer`
2. Set up Device, SwapChain, and RenderTargetView using SharpDX:

```csharp
using SharpDX;
using SharpDX.Direct3D11;
using SharpDX.DXGI;
using SharpDX.Mathematics.Interop;

public class OverlayRenderer
{
    private Device device;
    private SwapChain swapChain;
    private RenderTargetView renderTargetView;

    public void Initialize(IntPtr windowHandle)
    {
        var swapChainDesc = new SwapChainDescription
        {
            BufferCount = 1,
            ModeDescription = new ModeDescription(800, 600, new Rational(60, 1), Format.R8G8B8A8_UNorm),
            IsWindowed = true,
            OutputHandle = windowHandle,
            SampleDescription = new SampleDescription(1, 0),
            SwapEffect = SwapEffect.Discard,
            Usage = Usage.RenderTargetOutput
        };

        Device.CreateWithSwapChain(DriverType.Hardware, DeviceCreationFlags.None, swapChainDesc, out device, out swapChain);

        // Set up render target view
        var backBuffer = swapChain.GetBackBuffer<Texture2D>(0);
        renderTargetView = new RenderTargetView(device, backBuffer);
    }

    public void Render()
    {
        device.ImmediateContext.ClearRenderTargetView(renderTargetView, new RawColor4(0, 0, 0, 0));
        swapChain.Present(1, PresentFlags.None);
    }
}
```

> Note: The Render method clears the screen with transparency.

### Step 5: Add a Render Loop in OverlayForm

In OverlayForm.cs, add the following to start rendering:

```csharp
private OverlayRenderer renderer;

protected override void OnLoad(EventArgs e)
{
    base.OnLoad(e);
    renderer = new OverlayRenderer();
    renderer.Initialize(this.Handle);
    RenderLoop();
}

private async void RenderLoop()
{
    while (true)
    {
        renderer.Render();
        await Task.Delay(16);  // Approximately 60 FPS
    }
}
```

### Step 6: Access DLL from Python (Optional)

1. Install pythonnet in Python:

```bash
pip install pythonnet
```

2. In your Python script, load and use the DLL:

```python
import clr
clr.AddReference("PathToYourOverlayLibrary.dll")
from YourNamespace import OverlayRenderer

overlay = OverlayRenderer()
overlay.Initialize(window_handle)   # Pass the overlay window handle if needed
```

## Contributing
Feel free to submit issues and enhancement requests!

## License
[Your chosen license]

---
Created with ❤️ using SharpDX and C#
