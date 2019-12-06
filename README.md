# SysWhispers

SysWhispers helps with evasion by generating header/ASM files to let them make direct system calls.

All core syscalls are supported from Windows XP to 10. Example generated files available in `output` folder.  

## Introduction

Various security products place hooks in user-mode APIs which allow them to redirect execution flow to their engines and detect for suspicious behaviour. The functions in `ntdll.dll` that make the syscalls consist of just a few assembly instructions, so re-implementing them in your own implant can bypass the triggering of those security product hooks. This technique was popularized by [@Cn33liz](https://twitter.com/Cneelis) and his [blog post](https://outflank.nl/blog/2019/06/19/red-team-tactics-combining-direct-system-calls-and-srdi-to-bypass-av-edr/) has more technical details worth reading.

SysWhispers provides red teamers the ability to generate header/ASM pairs for any system call in the core kernel image (`ntoskrnl.exe`) across any Windows version starting from XP. The headers will also include the necessary type definitions.

The main implementation difference between this and the [Dumpert](https://github.com/outflanknl/Dumpert) POC is that this doesn't call `RtlGetVersion` to query the OS version, but instead does this in the assembly by querying the PEB directly. The benefit is being able to call one function that supports multiple Windows versions instead of calling multiple functions each supporting one version.

## Usage and Examples

```powershell
# Export all functions with compatibility for all supported Windows versions (see output dir).
py .\syswhispers.py --preset all -o syscalls_all

# Export just the common functions with compatibility for Windows 7, 8, and 10.
py .\syswhispers.py --preset common -o syscalls_common

# Export NtProtectVirtualMemory and NtWriteVirtualMemory with compatibility for all versions.
py .\syswhispers.py --functions NtProtectVirtualMemory,NtWriteVirtualMemory -o syscalls_mem

# Export all functions with compatibility for Windows 7, 8, and 10.
py .\syswhispers.py --versions 7,8,10 -o syscalls_78X
```

```
PS C:\Projects\SysWhispers> py .\syswhispers.py --preset common --out-file syscom

  ,         ,       ,_ /_   .  ,   ,_    _   ,_   ,
_/_)__(_/__/_)__/_/_/ / (__/__/_)__/_)__(/__/ (__/_)__
      _/_                         /
     (/                          /   @Jackson_T, 2019

SysWhispers: Why call the kernel when you can whisper?

Common functions selected.

Complete! Files written to:
        syscom.asm
        syscom.h
```

## Common Functions

Using the `--preset common` switch will create a header/ASM pair with the following functions:

- NtCreateProcess (CreateProcess)
- NtCreateThreadEx (CreateRemoteThread)
- NtOpenProcess (OpenProcess)
- NtOpenThread (OpenThread)
- NtSuspendProcess
- NtSuspendThread (SuspendThread)
- NtResumeProcess
- NtResumeThread (ResumeThread)
- NtGetContextThread (GetThreadContext)
- NtSetContextThread (SetThreadContext)
- NtClose (CloseHandle)
- NtReadVirtualMemory (ReadProcessMemory)
- NtWriteVirtualMemory (WriteProcessMemory)
- NtAllocateVirtualMemory (VirtualAllocEx)
- NtProtectVirtualMemory (VirtualProtectEx)
- NtFreeVirtualMemory (VirtualFreeEx)
- NtQuerySystemInformation (GetSystemInfo)
- NtQueryDirectoryFile
- NtQueryInformationFile
- NtQueryInformationProcess
- NtQueryInformationThread
- NtCreateSection (CreateFileMapping)
- NtOpenSection
- NtMapViewOfSection
- NtUnmapViewOfSection
- NtAdjustPrivilegesToken (AdjustTokenPrivileges)
- NtDeviceIoControlFile (DeviceIoControl)
- NtQueueApcThread (QueueUserAPC)
- NtWaitForMultipleObjects (WaitForMultipleObjectsEx)

## Importing into Visual Studio

TBD.

## Caveats and Limitations

- Only 64-bit Windows is supported at this time.
- System calls from the graphical subsystem (`win32k.sys`) are not supported.

## Credits

This script was developed by [@Jackson_T](https://twitter.com/Jackson_T) but builds upon the work of many others:

- [@j00ru](https://twitter.com/j00ru) for maintaining syscall numbers in machine-readable formats.
- [@FoxHex0ne](https://twitter.com/FoxHex0ne) for cataloguing many function prototypes and typedefs in a machine-readable format.
- [@PetrBenes](https://twitter.com/PetrBenes), [NTInternals.net team](https://undocumented.ntinternals.net/), and [MSDN](https://docs.microsoft.com/en-us/windows/) for additional prototypes and typedefs.
- [@Cn33liz](https://twitter.com/Cneelis) for the initial [Dumpert](https://github.com/outflanknl/Dumpert) POC implementation.

## Licence

This project is licensed under the Apache License 2.0.