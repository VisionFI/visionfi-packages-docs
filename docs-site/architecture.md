# Architecture

## Three-Layer Containment Model

Scout uses a layered containment architecture that separates your application, the execution runtime, and VisionFI's core logic into distinct security boundaries.

```
┌─────────────────────────────────────────────────┐
│  Your Application (C# / .NET)                   │
│  ├── VisionFI.Scout NuGet package               │
│  │   └── ScoutEngine.ReviewPolicyAsync()        │
│  │              ↓ native interop                 │
│  ├── Layer 2: Native Engine (bundled runtime)    │
│  │   ├── Sandboxed execution environment         │
│  │   ├── Host-controlled external access         │
│  │   ├── Authenticated outbound client          │
│  │   └── Async I/O management                   │
│  │              ↓ sandbox boundary               │
│  ├── Layer 3: Sealed Analysis Module             │
│  │   ├── Policy review logic                    │
│  │   ├── Prompt construction                    │
│  │   ├── Response parsing                       │
│  │   └── Zero external capabilities             │
│  │              ↓ host-mediated call             │
│  └── Managed Inference (operated by VisionFI)   │
└─────────────────────────────────────────────────┘
```

### Layer 1: C# NuGet Package (Your Interface)

- Clean .NET API: `ScoutEngine`, `PolicyDocument`, `ReviewResult`
- Handles async wrapping, logging, DI integration
- Native interop is an internal implementation detail

### Layer 2: Native Engine (Control Plane)

The bundled native library serves as the control plane within your environment. It is responsible for:

- Initializing and managing the sandboxed execution environment
- Loading the sealed analysis module
- Defining the controlled interface available to the sandboxed code
- Mediating all external I/O (managed inference requests)
- Managing async operations for long-running inference requests
- Exposing a clean interop surface to the .NET layer

### Layer 3: Sealed Analysis Module (VisionFI Intelligence)

VisionFI's proprietary analysis logic runs inside a sandboxed execution environment. This module:

- Runs fully sandboxed — **no filesystem, no network, no threads, no syscalls**
- Contains all business logic: prompt construction, response parsing, workflow orchestration
- Communicates with the outside world **exclusively** through host-controlled functions
- Cannot independently reach the network, access the filesystem, or exfiltrate data

## Sandbox Containment — What It Means

The sandbox enforces containment at the **runtime level**, not by convention. By default, the analysis module receives zero capabilities:

| Capability | Analysis Module Access | Controlled By |
|------------|----------------------|---------------|
| Network / HTTP | **None** — cannot establish connections | Native engine mediates all calls |
| Filesystem | **None** — no access to any files | Native engine controls all I/O |
| LLM Inference | Via host-controlled function only | Native engine owns the client and auth |
| Environment Variables | **None** | Native engine decides what to expose |
| Memory | Own isolated memory only | Sandbox runtime enforces boundaries |
| Threads | **None** — single-threaded | Native engine handles all concurrency |

This is **provable containment**. Your security team can verify that the analysis module physically cannot exfiltrate data because the runtime does not expose the capability.

## Managed Inference Flow

Since the sealed module has no network access, all managed inference is mediated by the host:

1. Analysis module constructs the prompt (combining your PDFs with the evaluation framework)
2. Module requests inference through a host-controlled function
3. Call crosses the sandbox boundary into the native engine
4. Engine makes an authenticated HTTPS call for managed inference
5. Analysis module is suspended (from its perspective, it's a synchronous call)
6. Response returns; engine passes the result back into the sandbox
7. Module parses the response and constructs the final output

## Data Flow

```
Your PDF bytes
  → base64 encoded by the SDK
  → passed into the sandboxed module's memory
  → module constructs the inference request
  → host-controlled function crosses sandbox boundary
  → native engine makes authenticated HTTPS call to LLM
  → response flows back through same path
  → module parses response into ReviewResult
  → result returned to your C# application
```

**What leaves your machine:** Only the managed inference request (your PDF content + the analysis prompt, sent over HTTPS through a VisionFI-managed channel).

**What stays on your machine:** Everything else — the Scout engine, the analysis logic, the results.

## Platform Support

The same sealed module works across platforms. Only the native engine binary changes per OS:

| Platform | Status |
|----------|--------|
| Windows x64 | Supported |
| macOS ARM (Apple Silicon) | Supported |
| macOS x64 | Supported |
| Linux x64 | Supported |
