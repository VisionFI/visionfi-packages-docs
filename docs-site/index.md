# VisionFI Scout SDK

**AI-powered loan policy review that runs in your environment, under your control.**

Scout analyzes your institution's loan policy and checklist documents against VisionFI's consumer-loan QC workflow requirements, producing a structured readiness assessment and draft `InstitutionConfig` — all without your data leaving your network.

---

## How It Works

```
Your Application (.NET / C#)
  ↓ NuGet package
Scout SDK (native binary — all processing happens locally)
  ↓ sealed Wasm sandbox
Policy Review Engine (VisionFI IP — you never see or touch it)
  ↓ controlled API call
LLM Inference (Anthropic Claude — only call that leaves your machine)
```

1. You pass in PDF documents (loan policy, checklist)
2. Scout's sealed engine analyzes them against 9 QC dimensions
3. You get back a structured report with a readiness verdict and draft configuration

**Your documents stay on your machine.** The only external call is to the Anthropic API for LLM inference — and that call is mediated and controlled by the Scout runtime.

---

## Quick Start

```csharp
using VisionFI.Scout;

var engine = new ScoutEngine(new ScoutOptions
{
    ApiKey = "your-anthropic-api-key"
});

var doc = PolicyDocument.FromFile("consumer-loan-policy.pdf");
var result = await engine.ReviewPolicyAsync(doc);

Console.WriteLine(result.MarkdownReport);
```

That's it. No cloud setup, no VisionFI account, no data sharing agreements.

[Get started in 5 minutes :material-arrow-right:](getting-started.md){ .md-button .md-button--primary }
[View architecture :material-arrow-right:](architecture.md){ .md-button }
