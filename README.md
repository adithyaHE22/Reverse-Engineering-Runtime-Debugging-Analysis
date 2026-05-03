# 🎮 Reverse Engineering and Runtime Manipulation of Legacy Game Software

> A hands-on reverse engineering analysis of **AssaultCube v1.2.0.2**, demonstrating how legacy game software can be analyzed and manipulated at the assembly level using static and dynamic analysis techniques.
>
> 📥 **[Click here to read the full report (PDF)](https://mysliit-my.sharepoint.com/:b:/g/personal/it23605398_my_sliit_lk/IQBOlvgYiSCpTryIrPD3Y_K7AcuX5yMT475l_ocn0MaKo5g?e=haIaIy)**
> 
> [▶️ Click here to watch the Gameplay Proof Video](https://mysliit-my.sharepoint.com/:v:/g/personal/it23605398_my_sliit_lk/IQC1p7_tjAPLSaHVpeOM4ACSAWgGaT7EXA7pMN-vSKj_5V8?nav=eyJyZWZlcnJhbEluZm8iOnsicmVmZXJyYWxBcHAiOiJPbmVEcml2ZUZvckJ1c2luZXNzIiwicmVmZXJyYWxBcHBQbGF0Zm9ybSI6IldlYiIsInJlZmVycmFsTW9kZSI6InZpZXciLCJyZWZlcnJhbFZpZXciOiJNeUZpbGVzTGlua0NvcHkifX0&e=lSrad9)

---

## 📌 Overview

This report documents a complete reverse engineering engagement targeting the legacy first-person shooter game **AssaultCube v1.2.0.2**. The analysis covers the full process from static disassembly to runtime memory manipulation — identifying how the player's health is stored, locating the exact assembly instruction responsible for health reduction, and patching it at runtime.

The exercise highlights critical security weaknesses found in legacy software and demonstrates why modern applications must implement strong anti-tampering and anti-reverse-engineering protections.

> 💡 *This report was originally created as a university assignment. It is shared here as a learning reference for reverse engineering and secure software concepts.*

---

## 🧪 Environment

| Component | Details |
|-----------|---------|
|  Target Application | AssaultCube v1.2.0.2 |
|  Executable | `ac_client.exe` |
|  Platform | Windows 10 64-bit |
|  Disassembler | IDA Free |
|  Debugger | x32dbg |
|  Modification Method | Runtime Assembly Patching (NOP) |

---

## 🔍 Methodology

The analysis followed a structured, phased approach:

```
Game Selection  →  Static Analysis  →  Dynamic Analysis
      →  Breakpoint Analysis  →  Instruction Modification  →  Verification
```

1. Target executable selected and prepared for analysis
2. Static analysis performed in **IDA Free** — strings, functions, assembly inspected
3. Gameplay-related strings (`health`, `onHit`, `SV_DAMAGE`) traced to relevant functions
4. Runtime debugging performed using **x32dbg** while the game was running
5. Memory searched and narrowed down by intentionally taking damage (`100 → 88 → 84`)
6. **Hardware breakpoint** (On Write) set on the health memory address
7. Exact instruction responsible for health reduction identified
8. Instruction replaced with **NOP** bytes at runtime
9. Gameplay verified — health no longer decreases after taking damage ✅

---

## 🔎 Static Analysis Findings

Using **IDA Free**, the following key strings were identified inside the executable:

| String | Relevance |
|--------|-----------|
| `health` | Points to health display/UI code |
| `onHit` | Leads to hit event handler function |
| `SV_DAMAGE` | Damage processing reference |
| `SV_GIBDAMAGE` | Extreme damage processing |
| `damageindicator` | On-screen damage effects |

The key assembly instructions found near the `onHit` function:

```asm
sub edi, eax         ; Subtract damage from EDI
sub [ebx+4], edi     ; Write reduced health back to player structure
```

- **EBX** → base address of the player structure  
- **[EBX+4]** → player's current health value  
- **EDI** → damage amount being applied

---

## ⚡ Dynamic Analysis Findings

Using **x32dbg** attached to the live game process:

- Memory searched for integer value `100` (initial health)
- Player intentionally damaged → health scanned again at new values
- Exact health memory address isolated after multiple scan rounds
- Hardware write breakpoint triggered on damage → execution paused automatically

**Identified instruction:**
```
Address: 00429D1F
Instruction: sub dword ptr ds:[ebx+4], edi
```

This confirmed:
```
Health = Health − Damage
```

---

## 🔧 Assembly-Level Modification

The identified subtraction instruction was patched **at runtime** using x32dbg:

| Before Patch | After Patch |
|---|---|
| `sub dword ptr ds:[ebx+4], edi` | `nop` / `nop` / `nop` (3 bytes) |

After patching, the game was resumed and the player took damage repeatedly — **health value remained unchanged**, confirming the modification was successful. 🎯

---

## 🚨 Security Weaknesses Identified

| Weakness | Description |
|----------|-------------|
| ❌ No Anti-Debugging | Debugger attached freely with no resistance |
| ❌ No Integrity Verification | Assembly patched at runtime with no detection |
| ❌ Plain Memory Storage | Health stored as a raw integer — easily found and edited |
| ❌ No Code Obfuscation | Assembly logic readable and straightforward |
| ❌ No Validation Checks | Manipulated values accepted without verification |

---

## 🛡️ Mitigation Strategies

| Protection | Purpose |
|------------|---------|
| Anti-Debugging | Detect and block debugger attachment |
| Integrity Checks | Detect unauthorized runtime code changes |
| Code Obfuscation | Make disassembly harder to interpret |
| Memory Encryption | Hide sensitive values like health and ammo |
| Validation Checks | Verify variable values before applying updates |

---

## 🛠️ Tools Used

### 🔬 Static Analysis
![IDA Free](https://img.shields.io/badge/IDA%20Free-Disassembler-4A90D9?style=for-the-badge&logo=windows&logoColor=white)

### 🐛 Dynamic Analysis & Debugging
![x32dbg](https://img.shields.io/badge/x32dbg-Debugger-CC0000?style=for-the-badge&logo=linux&logoColor=white)

### 🎮 Target Software
![AssaultCube](https://img.shields.io/badge/AssaultCube-v1.2.0.2-228B22?style=for-the-badge&logo=windows&logoColor=white)


---



## ⚠️ Disclaimer

This analysis was conducted in an **isolated local environment** for **educational purposes only**. The target application is a free, open-source game. No online systems, real users, or external services were affected. Unauthorized reverse engineering of commercial software may be illegal.

---

## 📄 License

Shared for **learning and reference purposes**. Please credit appropriately if you use any part of this work.
