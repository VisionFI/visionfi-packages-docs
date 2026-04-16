# VisionFI Scout SDK

**AI-powered loan policy review that runs in your environment, under your control.**

Scout analyzes your institution's loan policy and checklist documents against VisionFI's consumer-loan QC workflow requirements, producing a structured readiness assessment and draft `InstitutionConfig` — all without your data leaving your network.

---

## How It Works

```
Your Application (.NET / C#)
  ↓ NuGet package
Scout SDK (sealed native engine — all processing happens locally)
  ↓ secure sandbox
Policy Review Engine (VisionFI intelligence — sealed and tamper-proof)
  ↓ controlled, authenticated API call
LLM Inference (managed by VisionFI — you don't need your own AI account)
```

1. You pass in PDF documents (loan policy, checklist)
2. Scout's sealed engine analyzes them against 9 QC dimensions
3. You get back a structured report with a readiness verdict and draft configuration

**Your documents stay on your machine.** The only external call is for AI inference — authenticated with credentials VisionFI provides, routed through a secure channel.

---

## Quick Start

```csharp
using VisionFI.Scout;

var engine = new ScoutEngine(new ScoutOptions
{
    ApiKey = "your-visionfi-scout-key"  // provided by VisionFI during onboarding
});

var doc = PolicyDocument.FromFile("consumer-loan-policy.pdf");
var result = await engine.ReviewPolicyAsync(doc);

Console.WriteLine(result.MarkdownReport);
```

That's it. No cloud setup, no AI vendor accounts, no data sharing agreements.

[Get started in 5 minutes :material-arrow-right:](getting-started.md){ .md-button .md-button--primary }
[View architecture :material-arrow-right:](architecture.md){ .md-button }
