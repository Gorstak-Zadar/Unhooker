# 🔓 Unhooker26H1

> **Advanced Windows API Unhooking Tool for Windows 11 26H1**
> 
> Removes inline hooks, restores EAT/IAT/Syscall stubs, patches AMSI/ETW, and cleans memory artifacts.

![PowerShell](https://img.shields.io/badge/PowerShell-5.1+-5391FE?style=for-the-badge&logo=powershell&logoColor=white)
![Windows](https://img.shields.io/badge/Windows_11-26H1-0078D4?style=for-the-badge&logo=windows11&logoColor=white)
![License](https://img.shields.io/badge/License-Research_Only-red?style=for-the-badge)

---

## ⚠️ Disclaimer

```
This tool is intended for authorized security research, red team operations, 
and educational purposes only. Unauthorized use against systems you do not 
own or have explicit permission to test is illegal and unethical.

The author assumes no liability for misuse of this software.
```

---

## ✨ Features

### 🎯 Phase 1: Core DLL Unhooking

| DLL | JMP Unhook | Syscall Restore | EAT Unhook | IAT Unhook |
|-----|:----------:|:---------------:|:----------:|:----------:|
| `ntdll.dll` | ✅ | ✅ | ✅ | ➖ |
| `kernel32.dll` | ✅ | ➖ | ✅ | ✅ |
| `kernelbase.dll` | ✅ | ➖ | ✅ | ✅ |
| `advapi32.dll` | ✅ | ➖ | ✅ | ✅ |

### 🔧 Phase 2: Extended DLL Unhooking

| Category | DLLs |
|----------|------|
| 🔐 **Security** | `sechost.dll`, `sspicli.dll`, `cryptbase.dll` |
| 🖥️ **UI/Graphics** | `user32.dll`, `win32u.dll`, `gdi32.dll` |
| 🌐 **Network** | `ws2_32.dll`, `wininet.dll`, `winhttp.dll` |
| 🔒 **Crypto** | `crypt32.dll`, `bcrypt.dll`, `ncrypt.dll` |
| ⚙️ **Runtime** | `msvcrt.dll`, `ucrtbase.dll`, `combase.dll`, `rpcrt4.dll` |

### 🧹 Phase 3: Memory Cleanup

| Feature | Description |
|---------|-------------|
| 🧠 **PEB Cleanup** | Clears `BeingDebugged` flag and `NtGlobalFlag` |
| 📦 **Heap Flags** | Removes debug heap indicators |
| 📞 **Kernel Callback Table** | Verifies KCT integrity |

### 🛡️ Phase 4: Security Patches

| Bypass | Target | Functions Patched |
|--------|--------|-------------------|
| 🦠 **AMSI** | `amsi.dll` | `AmsiScanBuffer`, `AmsiScanString`, `AmsiOpenSession` |
| 📊 **ETW** | `ntdll.dll` | `EtwEventWrite`, `EtwEventWriteFull`, `EtwEventWriteEx`, `EtwEventWriteTransfer`, `NtTraceEvent` |
| 🔭 **Defender** | `mpclient.dll` | `MpManagerOpen` |
| 📝 **Script Logging** | `ntdll.dll` | `NtTraceControl` |
| 📡 **Sensor** | `sensapi.dll` | `IsNetworkAlive` |

---

## 🚀 Quick Start

### Basic Usage

```powershell
# Run with PowerShell (requires Administrator)
.\Unhooker26H1.ps1
```

### Expected Output

```
[--------------------------------------------------]
[  SharpUnhooker26H1 - Windows 11 26H1 Unhooker   ]
[           Adapted for Build 26xxx               ]
[--------------------------------------------------]

[*] Checking system security features...
[*] CFG (Control Flow Guard) is ENABLED
[*] VBS (Virtualization-Based Security) is DISABLED

[*] Phase 1: Core DLL Unhooking...
[+++] NTDLL.DLL IS UNHOOKED (JMP)!
[+++] RESTORED 47 SYSCALL STUBS IN NTDLL.DLL!
[+++] NTDLL.DLL EXPORTS ARE CLEANSED!
[+++] KERNEL32.DLL IS UNHOOKED (JMP)!
[+++] KERNEL32.DLL EXPORTS ARE CLEANSED!
[+++] KERNEL32.DLL IMPORTS ARE CLEANSED!
...

[*] Phase 2: Extended DLL Unhooking...
[+++] SECHOST.DLL IS UNHOOKED (JMP)!
...

[*] Phase 3: Memory Cleanup...
[+++] PEB CLEANED!
[+++] HEAP FLAGS CLEANED!
[+++] KERNEL CALLBACK TABLE CHECKED!

[*] Phase 4: Security Patches...
[+++] AMSI SUCCESSFULLY PATCHED!
[+++] ETW SUCCESSFULLY PATCHED (26H1 Enhanced)!
[+++] DEFENDER TELEMETRY PATCHED!
[+++] SCRIPT BLOCK LOGGING PATCHED!

[--------------------------------------------------]
[           UNHOOKING COMPLETE!                    ]
[--------------------------------------------------]
```

---

## 🔬 How It Works

### 1️⃣ JMP Unhooking (Inline Hook Removal)

Security products patch function prologues with `JMP` instructions to redirect execution:

```
┌─────────────────────────────────────────────────────┐
│  Original (Clean)          Hooked                   │
│  ─────────────────         ──────                   │
│  4C 8B D1                  E9 XX XX XX XX           │
│  B8 55 00 00 00            (jmp <hook_address>)     │
│  (mov r10, rcx)                                     │
│  (mov eax, 55h)                                     │
└─────────────────────────────────────────────────────┘
```

**Solution:** Read original `.text` section from disk and overwrite in-memory copy.

```
     Disk (Clean)                    Memory (Hooked)
    ┌─────────────┐                 ┌─────────────┐
    │  ntdll.dll  │ ═══════════════►│  ntdll.dll  │
    │   .text     │   Overwrite     │   .text     │
    └─────────────┘                 └─────────────┘
```

### 2️⃣ Syscall Stub Restoration

Detects hooked `Nt*` functions and restores original syscall stubs:

```asm
; Original x64 syscall stub
mov r10, rcx          ; 4C 8B D1
mov eax, <syscall#>   ; B8 XX XX XX XX
syscall               ; 0F 05
ret                   ; C3
```

### 3️⃣ EAT Unhooking

Compares Export Address Table RVAs against disk copy and restores any modifications.

### 4️⃣ IAT Unhooking

Re-resolves import addresses and compares against Import Address Table entries.

### 5️⃣ AMSI Patch

```asm
; Patches AmsiScanBuffer to return AMSI_RESULT_CLEAN
mov eax, 0x80070057    ; B8 57 00 07 80
ret                    ; C3
```

### 6️⃣ ETW Patch

```asm
; Patches EtwEventWrite to return success without logging
xor rax, rax           ; 48 33 C0
ret                    ; C3
```

---

## 🏗️ Architecture

```
┌────────────────────────────────────────────────────────────┐
│                    Unhooker26H1.ps1                        │
├────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   PEReader   │  │   Dynavoke   │  │ PatchAMSI    │     │
│  │              │  │              │  │ AndETW26H1   │     │
│  │ • DOS Header │  │ • GetExport  │  │              │     │
│  │ • NT Headers │  │ • NtProtect  │  │ • PatchAMSI  │     │
│  │ • Sections   │  │ • GetPEB     │  │ • PatchETW   │     │
│  │ • Exports    │  │ • NtWrite    │  │ • Telemetry  │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                            │
│  ┌─────────────────────────────────────────────────────┐  │
│  │              SharpUnhooker26H1                      │  │
│  │                                                     │  │
│  │  • JMPUnhooker()      • EATUnhooker()              │  │
│  │  • SyscallUnhooker()  • IATUnhooker()              │  │
│  │  • PEBCleanup()       • HeapFlagsCleanup()         │  │
│  │  • KernelCallbackTableCleanup()                    │  │
│  │  • CheckCFGStatus()   • CheckVBSStatus()           │  │
│  └─────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────┘
```

---

## 🛡️ Protected Functions

These functions are **blacklisted** to prevent system instability:

```
EnterCriticalSection    LeaveCriticalSection    DeleteCriticalSection
InitializeSListHead     HeapAlloc               HeapReAlloc
HeapSize                HeapFree                RtlEnterCriticalSection
RtlLeaveCriticalSection RtlDeleteCriticalSection RtlAllocateHeap
RtlReAllocateHeap       RtlFreeHeap             RtlSizeHeap
NtClose                 RtlExitUserThread       LdrShutdownThread
RtlUserThreadStart      BaseThreadInitThunk     KiUserApcDispatcher
KiUserCallbackDispatcher KiUserExceptionDispatcher LdrInitializeThunk
```

---

## 📋 Requirements

| Requirement | Details |
|-------------|---------|
| 🖥️ **OS** | Windows 11 26H1 (Build 26xxx) |
| ⚡ **PowerShell** | 5.1 or higher |
| 🔑 **Privileges** | Administrator (Run as Admin) |
| 🏗️ **.NET** | Framework 4.0+ (built-in) |

---

## 🔍 Security Feature Detection

The tool automatically detects and reports:

| Feature | Impact |
|---------|--------|
| **CFG** (Control Flow Guard) | May limit some unhooking techniques |
| **VBS** (Virtualization-Based Security) | Kernel-level protections may persist |
| **HVCI** (Hypervisor Code Integrity) | Some syscalls may remain protected |

---

## 📁 Project Structure

```
Unhooker/
├── 📄 Unhooker.ps1      # Main script - Windows 11 26H1 version (72.7 KB)
└── 📄 README.md         # This documentation
```

---

## 🔗 References

| Resource | Description |
|----------|-------------|
| [D/Invoke](https://github.com/TheWover/DInvoke) | Dynamic invocation techniques |
| [SharpUnhooker](https://github.com/GetRektBoy724/SharpUnhooker) | Original inspiration |
| [Red Team Notes](https://www.ired.team/) | AMSI/ETW bypass research |
| [MSDN PE Format](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format) | PE structure documentation |

---

## 📜 Changelog

### v2.0 - 26H1 Edition

- ✅ Added support for Windows 11 26H1 (Build 26xxx)
- ✅ Extended DLL coverage (20 DLLs)
- ✅ Syscall stub restoration for ntdll
- ✅ Enhanced ETW patching (5 functions)
- ✅ Enhanced AMSI patching (3 functions)
- ✅ PEB/Heap cleanup
- ✅ Kernel Callback Table verification
- ✅ CFG/VBS detection
- ✅ Defender telemetry patching
- ✅ Script block logging bypass
- ✅ Expanded blacklist for stability

---

## 👤 Author

**Gorstak**

---

<p align="center">
  <img src="https://img.shields.io/badge/Made_with-❤️-red?style=for-the-badge" alt="Made with love"/>
  <br><br>
  <sub>🔬 Built for security research • 🛡️ Use responsibly</sub>
</p>
