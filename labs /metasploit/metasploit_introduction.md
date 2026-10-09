---
# 🚀 Metasploit Framework: Comprehensive Reference Guide

## 1. Introduction & Core Versions

Metasploit is the world's most widely used exploitation framework, supporting all phases of a penetration testing engagement from initial information gathering to post-exploitation.

- **Metasploit Pro:** The commercial version featuring a Graphical User Interface (GUI) and task automation features.
- **Metasploit Framework:** The open-source, command-line-driven version predominantly used in security testing distributions and AttackBox environments.

---

## 2. Fundamental Terminology

Before interacting with modules, it is vital to understand three core concepts:

- **Vulnerability:** A design, coding, or logic flaw in a target system that can expose confidential data or permit unauthorized code execution.
- **Exploit:** A piece of code that takes advantage of a specific target vulnerability.
- **Payload:** The code delivered *by* the exploit to run on the target system to achieve the desired outcome (e.g., open a shell, launch a program).

---

## 3. Main Components & Modules

Metasploit organizes its functionality into specialized modules and standalone tools:

- **Auxiliary:** Scanners, fuzzers, crawlers, and sniffers used for information gathering and non-exploitation tasks.
- **Exploits:** Organized by target operating system/service; pieces of code designed to leverage specific vulnerabilities.
- **Payloads:** Code executed on the target. Divided into:
- *Singles (Inline):* Self-contained payloads that do not require external downloads (e.g., `generic/shell_reverse_tcp`).
- *Stagers:* Small code pieces designed to set up a connection channel and download a larger payload.
- *Stages:* The larger payload components downloaded by stagers (indicated by a `/` instead of an `_` in naming conventions, e.g., `windows/x64/shell/reverse_tcp`).
- *Adapters:* Wrappers that convert single payloads into alternative formats (like PowerShell commands).
- **Post:** Modules used during the post-exploitation phase to gather data or pivot across a network.
- **Encoders / Evasion:** Tools to encode payloads or attempt to bypass security protections/antivirus signatures.
- **NOPs:** No-Operation instructions used as padding/sleds to stabilize buffer sizes.
- **Standalone Tools:** Utilities like `msfvenom` for payload generation.

---

## 4. The Metasploit Console (`msfconsole`) & Navigation

Launched via the `msfconsole` terminal command, the console operates on a **context-driven** system.

### Useful Console Commands & Features

- **`ls`, `ping`, `clear`, `history`:** Standard Linux utilities supported directly within the console. *(Note: Output redirection like `>` is not supported).*
- **Tab Completion:** Automatically completes commands and module paths.
- **`search <term>`:** Searches the module database (supports filters like `type:auxiliary` or `cve:2009`).
- **`use <module>`:** Enters the specific context of a module (prompt changes to reflect the active module).
- **`info`:** Displays deep technical details, authors, and references for a module.
- **`back`:** Exits the current module context back to the main `msf6 >` prompt.

---

## 5. Configuring Parameters (`show options`)

Once inside a module context, you must configure required variables before running it.

- **`show options`:** Lists required and optional parameters for the active module.

### Common Parameters

- **`RHOSTS`:** Remote Host(s) — target IP address, IP range, CIDR notation (`/24`), or target file list (`file:/path/txt`).
- **`RPORT`:** Remote Port — the target service port (e.g., `445` for SMB).
- **`PAYLOAD`:** The specific payload attached to the exploit.
- **`LHOST`:** Local Host — your attacking machine's IP address.
- **`LPORT`:** Local Port — the listening port on your machine for reverse connections.
- **`SESSION`:** Active session ID used by post-exploitation modules.

### Parameter Management Commands

- `set PARAMETER_NAME VALUE` — Sets a variable within the current module context.
- `setg PARAMETER_NAME VALUE` — Sets a **global** variable across all modules until cleared or session ends.
- `unset PARAMETER_NAME` — Clears a specific parameter value.
- `unset all` — Clears all configured parameters in the current context.
- `unsetg PARAMETER_NAME` — Clears a global variable.

---

## 6. Execution, Verification, & Session Management

### Running Modules

- **`exploit`** or **`run`**: Launches the configured module.
- **`exploit -z`**: Runs the exploit and automatically backgrounds the resulting session immediately.
- **`check`**: Safely checks if a target system is vulnerable *without* executing the exploit (supported by select modules).

### Managing Active Sessions

- **`background`** (or `CTRL + Z`): Backgrounds an active session/Meterpreter prompt and returns you to the Metasploit console.
- **`sessions`**: Lists all active compromise sessions and connection details.
- **`sessions -i <ID>`**: Resumes direct interaction with a specific session (e.g., `sessions -i 2`).
