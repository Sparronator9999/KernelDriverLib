# KernelDriverLib - C# Kernel Driver Library

A C# (.NET) library for installing and accessing Windows kernel drivers.

## Disclaimers

- This library can do some pretty powerful stuff (as it interfaces directly with kernel drivers). **Only use if you know what you're doing!**
- Linux will never be supported by this library, so don't ask.

## Features

This library can:

- Install and uninstall kernel drivers (.sys files)
  - Persistent install is not well supported yet.
- Issue commands to drivers via [DeviceIoControl](https://learn.microsoft.com/en-us/windows/win32/api/ioapiset/nf-ioapiset-deviceiocontrol).
  - ReadFile/WriteFile not supported yet.
- Restrict access of a driver's device (e.g. for WinRing0: `\\.\WinRing0_1_2_0`) to admin access only
  - Useful for reducing the attack surface of vulnerabilities in WinRing0 and other drivers

This library may be useful for interoping with [WinRing0](https://github.com/GermanAizek/WinRing0) or other similar drivers.

## Usage notes

This library's `Driver` class is designed to be extended by adding driver-specific IOCTLS to a subclass. For example:

```cs
public class Ring0Driver : Driver
{
    public Ring0Driver(string name) : base(name) { }
    public Ring0Driver(string name, string path) : base(name, path) { }

    public bool IOControl(Ring0IOCTL ctlCode)
    {
        return IOControl((uint)ctlCode);
    }

    public bool IOControl<TIn, TOut>(Ring0IOCTL ctlCode, ref TIn inBuffer, out TOut outBuffer)
        where TIn : unmanaged
        where TOut : unmanaged
    {
        return IOControl((uint)ctlCode, ref inBuffer, out outBuffer);
    }

    // add more overrides to your driver subclass as needed
}

public enum Ring0IOCTL : uint
{
    GetDriverVersion = 40000u << 16 | 0x800 << 2,
    GetRefCount      = 40000u << 16 | 0x801 << 2,
    ReadIOPortByte   = 40000u << 16 | 0x833 << 2 | 1 << 14,
    WriteIOPortByte  = 40000u << 16 | 0x836 << 2 | 2 << 14,
    // ...
}

```

Struct support hasn't been well tested, but should work if the `[StructLayout(LayoutKind.Sequential, Pack = 1)]` attribute is added to structs to be passed to the driver.

## Download

Development builds are available through [GitHub Actions](https://github.com/Sparronator9999/KernelDriverLib/actions).

Alternatively, if you don't have a GitHub account, you can download the latest build from [nightly.link](https://nightly.link/Sparronator9999/KernelDriverLib/workflows/build/main?preview).

(You probably want the `Release` build, unless you're debugging issues with the program)

Alternatively, you can [build the program yourself](#build).

## Build

### Using Visual Studio

1.  Install Visual Studio 2022 with the `.NET Desktop Development` workload checked.
2.  Download the code repository, or clone it with `git`.
3.  Extract the downloaded code, if needed.
4.  Open `KernelDriverLib.sln` in Visual Studio.
5.  Click `Build` > `Build Solution` to build everything.
6.  Your output, assuming default build settings, is located in `KernelDriverLib\bin\Debug\net48\`.
7.  ???
8.  Profit!

### From the command line

1.  Follow steps 1-3 above to install Visual Studio and download the code.
2.  Open `Developer Command Prompt for VS 2022` and `cd` to your project directory.
3.  Run `msbuild /t:restore` to restore the solution, including NuGet packages.
4.  Run `msbuild KernelDriverLib.sln /p:platform="Any CPU" /p:configuration="Debug"` to build
    the project, substituting `Debug` with `Release` (or `Any CPU` with `x86` or `x64`) as 
5.  Your output should be located in `KernelDriverLib\bin\Debug\net48\` (or similar).
6.  ???
7.  Profit!

## Supported Windows versions

In theory, any version of Windows that can run .NET Framework 4.8 (i.e. Windows 7 SP1 and later) can run this library.

In practice, only Windows 10 64-bit has been tested. Other versions ***should*** work, but haven't been tested.

## License and Copyright

Copyright © 2024 Sparronator9999.

This program is free software: you can redistribute it and/or modify it under
the terms of the GNU General Public License as published by the Free Software
Foundation, either version 3 of the License, or (at your option) any later
version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY
WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE. See the [GNU General Public License](LICENSE.md) for more
details.

