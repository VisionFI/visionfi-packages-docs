# VisionFI Policy Analyzer

**Create Scout CEL rule bundles from policy documents inside the partner environment.**

VisionFI Policy Analyzer is a .NET package for partners who do not work inside VisionFI CRM. A partner application sends consumer-lending policy PDFs or free-form policy text into the SDK, receives an evidence report and a staged CEL rule-bundle payload, and stores that payload for later Scout HQ integration.

Scout HQ provides the FI identity, authoring profile, schema artifacts, and provider credentials. Policy documents and policy text are not sent to Scout HQ.

---

## What It Produces

The SDK returns the same rule-bundle wrapper shape used by VisionFI CRM:

```json
{
  "product_key": "consumer-qc",
  "workflow_key": "consumer-loan-qc",
  "version": "2026.06.03.1",
  "bundle_json": {
    "institution_id": "fi-123",
    "version": "2026.06.03.1",
    "product_id": "consumer-loan-qc",
    "rules": [],
    "derivations": [],
    "tables": {}
  },
  "active": false
}
```

The payload is staged output. A partner-owned approval and storage workflow decides when and how the bundle is later integrated with Scout HQ.

## How It Works

```text
Partner .NET Application
  -> VisionFI Policy Analyzer
  -> Scout HQ using FI token
       - validates token
       - returns institution identity
       - returns authoring profile
       - returns provider credentials
  -> Local SDK authoring run
       - sends source material directly to managed inference
       - validates and shapes the CEL bundle
  -> Partner stores staged bundle
```

1. The partner application provides an FI token issued by Scout HQ.
2. The SDK retrieves a product/workflow-specific authoring profile from Scout HQ.
3. The SDK sends PDFs or free-form policy text to managed inference from the partner environment.
4. The SDK returns an evidence report and a CRM-compatible rule-bundle wrapper.

## Quick Start

```csharp
using VisionFI.Scout.Authoring;

var client = new ScoutAuthoringClient(new ScoutAuthoringOptions
{
    HqBaseUrl = "https://scout-hq.example.com",
    ScoutToken = "<fi-token>"
});

var result = await client.AuthorRuleBundleAsync(new AuthorRuleBundleRequest
{
    ProfileKey = "consumer-qc.consumer-loan-qc.policy-cel-authoring",
    Version = "2026.06.03.1",
    Sources =
    [
        AuthoringSource.FromPdf("consumer-loan-policy.pdf"),
        AuthoringSource.FromText("loan-checklist.txt", checklistText)
    ]
});

Console.WriteLine(result.EvidenceReportMarkdown);
Console.WriteLine(result.RuleBundleWrapperJson);
```

[Get started :material-arrow-right:](getting-started.md){ .md-button .md-button--primary }
[View architecture :material-arrow-right:](architecture.md){ .md-button }
