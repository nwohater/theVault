---
id: fa74ef61-b784-4afc-8e80-553aca175845
title: Getting Started
author: ai
tags:
- guide
- setup
- development
created: 2026-04-07T15:17:02.731468600Z
modified: 2026-04-07T15:17:02.731468600Z
project: WBMgr
status: draft
ai_tool: claude-code
canonical: false
protected: false
---

# Getting Started

This guide walks through setting up a development environment for WBMgr from scratch on Windows.

---

## Prerequisites

### 1. Node.js
Node.js v24 is already installed. Verify with:
```bash
node --version
```

### 2. Rust
Tauri requires the Rust toolchain. Install from [rustup.rs](https://rustup.rs):
```powershell
winget install Rustlang.Rustup
# or download and run the installer from https://rustup.rs
```

After installation, restart your terminal and verify:
```bash
rustc --version
cargo --version
```

### 3. Microsoft C++ Build Tools
Required on Windows for compiling Rust crates. Install via Visual Studio Installer — select the **"Desktop development with C++"** workload.

Alternatively, install the standalone Build Tools:
```powershell
winget install Microsoft.VisualStudio.2022.BuildTools
```

### 4. WebView2 Runtime
Tauri 2 on Windows uses WebView2. It is pre-installed on Windows 10/11. If missing, download from [Microsoft's site](https://developer.microsoft.com/microsoft-edge/webview2/).

---

## Installation

```bash
# Clone the repo (or navigate to it)
cd C:\Users\brand\Documents\src\wbmgr

# Install Node dependencies
npm install
```

---

## Running in Development

```bash
npm run tauri dev
```

This command:
1. Starts the Vite dev server on `http://localhost:1420`
2. Compiles the Rust backend (slow on first run — subsequent builds are incremental)
3. Opens a native Tauri window pointed at the Vite server

Hot module replacement (HMR) works for React/TypeScript changes. Rust changes require a full recompile.

---

## Building for Production

```bash
npm run tauri build
```

Outputs a native installer to `src-tauri/target/release/bundle/`. On Windows this produces an `.msi` and/or `.exe` installer.

---

## Frontend-Only Development

If you don't have Rust installed yet, you can develop the UI in a browser:
```bash
npm run dev
```
Then open `http://localhost:1420` in your browser. The Tauri APIs won't be available, but since all app logic is in React with no Tauri IPC calls (except the unused `greet` command), the full UI works in a browser.
