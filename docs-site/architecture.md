# Architecture

## Three-Layer Containment Model

Scout uses a layered containment architecture that separates your application, the execution runtime, and VisionFI's core logic into distinct security boundaries.

```
┌─────────────────────────────────────────────────┐
│  Your Application (C# / .NET)                   │
│  ├── VisionFI.Scout NuGet package               │
│  │   └── ScoutEngine.ReviewPolicyAsync()        │
│  │              ↓ P/Invoke                      │
│  ├── Layer 2: Native Rust Wrapper (.dll/.dylib) │
│  │   ├── Wasmtime runtime (embedded)            │
│  │   ├── Host-imported functions                │
│  │   ├── HTTP client (Anthropic API)            │
│  │   └── Tokio async runtime                    │
│  │              ↓ Wasm sandbox boundary          │
│  ├── Layer 3: Wasm Guest Module (.wasm)         │
│  │   ├── Policy review logic                    │
│  │   ├── Prompt construction                    │
│  │   ├── Response parsing                       │
│  │   └── Zero external capabilities             │
│  │              ↓ host import call               │
│  └── LLM Inference (Anthropic Claude API)       │
└─────────────────────────────────────────────────┘
```

### Layer 1: C# NuGet Package (Your Interface)

- Clean .NET API: `ScoutEngine`, `PolicyDocument`, `ReviewResult`
- Handles async wrapping, logging, DI integration
- P/Invoke calls to the native library are internal implementation details

### Layer 2: Rust Wrapper (Control Plane)

The native library (`.dll` on Windows, `.dylib` on macOS, `.so` on Linux) serves as the control plane. It is responsible for:

- Embedding and initializing the Wasmtime WebAssembly runtime
- Loading the sealed Wasm module
- Defining host-imported functions available to the Wasm guest
- Mediating all external I/O (LLM API calls)
- Managing async operations via Tokio for long-running LLM calls
- Exposing a C-compatible FFI surface

### Layer 3: Wasm Guest Module (Sealed Logic)

VisionFI's proprietary logic, compiled from Rust to WebAssembly (`wasm32-wasip1`). This module:

- Runs fully sandboxed — **no filesystem, no network, no threads, no syscalls**
- Contains all business logic: prompt construction, response parsing, workflow orchestration
- Communicates with the outside world **exclusively** through host-imported functions
- Cannot independently reach the network, access the filesystem, or exfiltrate data

## Wasm Containment — What It Means

The Wasm sandbox enforces containment at the **runtime level**, not by convention. By default, the guest module receives zero capabilities:

| Capability | Wasm Guest Access | Controlled By |
|------------|-------------------|---------------|
| Network / HTTP | **None** — cannot establish connections | Rust wrapper mediates all API calls |
| Filesystem | **None** — no access to any files | Rust wrapper controls all I/O |
| LLM Inference | Via host import only | Rust wrapper owns HTTP client and auth |
| Environment Variables | **None** | Rust wrapper decides what to expose |
| Memory | Own linear memory only | Wasmtime enforces boundaries |
| Threads | **None** — single-threaded | Rust wrapper handles all concurrency |

This is **provable containment**. Your security team can verify that the module physically cannot exfiltrate data because the runtime does not expose the capability.

## LLM Call Flow

Since the Wasm guest has no network access, all LLM inference is mediated by the host:

1. Wasm guest constructs the prompt (combining your PDFs with the analysis framework)
2. Guest calls `request_llm_inference` — a host-imported function
3. Call crosses the sandbox boundary into the Rust wrapper
4. Wrapper makes an async HTTPS call to the Anthropic API
5. Wasm module is suspended (from its perspective, it's a synchronous call)
6. Response returns; wrapper passes the result back into Wasm memory
7. Guest parses the response and constructs the final output

## Data Flow

```
Your PDF bytes
  → base64 encoded by C# SDK
  → passed into Wasm guest memory
  → guest constructs Anthropic API request body
  → host import call crosses sandbox boundary
  → Rust wrapper makes HTTPS call to Anthropic
  → response flows back through same path
  → guest parses response into ReviewResult
  → result returned to your C# application
```

**What leaves your machine:** Only the Anthropic API call (your PDF content + the analysis prompt, sent over HTTPS to Anthropic's API).

**What stays on your machine:** Everything else — the Scout engine, the analysis logic, the results.

## Platform Support

The same Wasm module and Rust wrapper work across platforms. Only the outermost layer changes:

| Platform | Native Library | NuGet RID |
|----------|---------------|-----------|
| Windows x64 | `scout_wrapper.dll` | `win-x64` |
| macOS ARM | `libscout_wrapper.dylib` | `osx-arm64` |
| macOS x64 | `libscout_wrapper.dylib` | `osx-x64` |
| Linux x64 | `libscout_wrapper.so` | `linux-x64` |
