# Security & Data Sovereignty

## The Core Guarantee

**Your data never touches VisionFI infrastructure.**

Scout runs entirely within your environment. There is no VisionFI cloud service, no VisionFI API endpoint, no data pipeline back to VisionFI. The only external call is to the Anthropic API for LLM inference — and that call is made from your machine, using your API key, under your control.

## What This Means for Your Organization

### No Data Processor Liability

VisionFI never acts as a data processor for your member/customer data. The loan policy documents you analyze are read by the Scout SDK running on your hardware and sent directly to the Anthropic API by code running in your environment.

### Simplified Vendor Due Diligence

Traditional AI vendor assessments ask: Where is my data stored? Who has access? What are the retention policies? With Scout, the answer to all of these is: **your existing policies apply, because the data never leaves your security perimeter.**

### No BAA / DPA Scope Expansion

Because VisionFI never sees, stores, or processes your data, there is no need to expand Business Associate Agreements or Data Processing Agreements to cover VisionFI for this data flow.

## The Anthropic API Call

The one external call Scout makes is to the Anthropic Messages API. Here's what you should know:

- **What's sent:** Your PDF document content (base64-encoded) + the analysis prompt
- **Where it goes:** `api.anthropic.com` over HTTPS (TLS 1.2+)
- **Authentication:** Your Anthropic API key (you control it)
- **Data retention:** Subject to [Anthropic's API data policy](https://www.anthropic.com/policies) — API inputs are not used for model training
- **You can audit it:** The request is made by the native library on your machine — your network monitoring tools see it

!!! info "Your Anthropic Relationship"
    You hold the direct relationship with Anthropic. VisionFI is not an intermediary. You can negotiate enterprise terms, data processing agreements, or zero-retention policies directly with Anthropic.

## IP Protection

VisionFI's analysis logic (the prompt engineering, evaluation framework, and response parsing) is compiled into a WebAssembly binary that runs inside a sandboxed runtime. A partner who inspects the native library cannot extract, modify, or reverse-engineer the analysis methodology — it exists only as Wasm bytecode.

## Containment Verification

The Wasm containment model is verifiable by your security team:

1. **The Wasm module has no network imports** — inspect the module's import table to confirm no socket, HTTP, or network functions are imported
2. **The Wasm module has no filesystem imports** — the WASI context is configured with zero preopened directories
3. **All external calls go through host imports** — the only bridge to the outside world is the `request_llm_inference` function, which the Rust wrapper controls
4. **The Wasm runtime enforces memory isolation** — the guest cannot read or write memory outside its own linear memory space

## Summary

| Concern | Status |
|---------|--------|
| Does VisionFI see my data? | **No** — Scout runs in your environment |
| Does VisionFI host anything? | **No** — no cloud service, no API endpoint |
| Where does my data go? | **Anthropic API only** — from your machine, with your key |
| Can VisionFI access my results? | **No** — results stay in your application's memory |
| Can the analysis logic be extracted? | **No** — sealed in a Wasm binary |
| Can I audit the network calls? | **Yes** — standard HTTPS from your machine |
| Do I need a DPA with VisionFI for data? | **No** — VisionFI never processes your data |
