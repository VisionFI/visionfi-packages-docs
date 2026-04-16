# Security & Data Sovereignty

## The Core Guarantee

**Your data never touches VisionFI infrastructure.**

Scout runs entirely within your environment. There is no VisionFI cloud service, no VisionFI API endpoint, no data pipeline back to VisionFI. The only external call is for LLM inference — made from your machine, authenticated with credentials VisionFI provides and manages on your behalf.

## What This Means for Your Organization

### No Data Processor Liability

VisionFI never acts as a data processor for your member/customer data. The loan policy documents you analyze are read by the Scout SDK running on your hardware. The LLM inference call goes directly from your environment to the AI provider.

### Simplified Vendor Due Diligence

Traditional AI vendor assessments ask: Where is my data stored? Who has access? What are the retention policies? With Scout, the answer to all of these is: **your existing policies apply, because the data never leaves your security perimeter** (except for the LLM inference call, which VisionFI manages).

### No BAA / DPA Scope Expansion

Because VisionFI never sees, stores, or processes your data, there is no need to expand Business Associate Agreements or Data Processing Agreements to cover VisionFI for this data flow.

## LLM Inference

The one external call Scout makes is for AI inference. Here's what you should know:

- **What's sent:** Your PDF document content + the analysis prompt, over HTTPS
- **Authentication:** Managed by VisionFI — you don't need your own AI vendor account
- **Data retention:** VisionFI selects AI providers with API-appropriate data policies — inference inputs are not used for model training
- **You can audit it:** The request is made from your machine — your network monitoring tools see it

!!! info "VisionFI Manages the AI Relationship"
    VisionFI handles the AI provider relationship, model selection, and authentication. You don't need to set up accounts with AI vendors, negotiate data terms, or manage API keys with third parties. Your Scout API key is all you need.

## IP Protection

VisionFI's analysis logic (the evaluation framework, prompt engineering, and response parsing) is compiled into a sealed binary that runs inside a sandboxed runtime. The analysis methodology cannot be extracted, modified, or reverse-engineered from the distributed package.

## Containment Verification

The containment model is verifiable by your security team:

1. **The analysis module has no network access** — it runs in a sandbox with zero network capabilities
2. **The analysis module has no filesystem access** — the sandbox has no access to your disk
3. **All external calls go through host-controlled functions** — the only bridge to the outside world is the inference function, which the native engine controls
4. **Memory isolation is enforced** — the analysis module cannot read or write memory outside its own isolated address space

## Summary

| Concern | Status |
|---------|--------|
| Does VisionFI see my data? | **No** — Scout runs in your environment |
| Does VisionFI host anything? | **No** — no cloud service, no API endpoint |
| Do I need an AI vendor account? | **No** — VisionFI manages the AI relationship |
| Where does my data go? | **LLM inference call only** — from your machine |
| Can VisionFI access my results? | **No** — results stay in your application's memory |
| Can the analysis logic be extracted? | **No** — sealed in a sandboxed binary |
| Can I audit the network calls? | **Yes** — standard HTTPS from your machine |
| Do I need a DPA with VisionFI for data? | **No** — VisionFI never processes your data |
