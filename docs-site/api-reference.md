# API Reference

This page documents the public surface for VisionFI Policy Analyzer.

The NuGet package ID is `VisionFI.PolicyAnalyzer`. The current public C# namespace is `VisionFI.Scout.Authoring`.

## ScoutAuthoringClient

Main entry point for rule-bundle authoring.

```csharp
var client = new ScoutAuthoringClient(new ScoutAuthoringOptions
{
    HqBaseUrl = "https://scout-hq.example.com",
    ScoutToken = "<fi-token>"
});
```

### `AuthorRuleBundleAsync`

```csharp
Task<AuthorRuleBundleResult> AuthorRuleBundleAsync(
    AuthorRuleBundleRequest request,
    CancellationToken cancellationToken = default);
```

Authors a staged CEL rule bundle from PDF and text source material.

## ScoutAuthoringOptions

| Property | Type | Description |
|----------|------|-------------|
| `HqBaseUrl` | `string` | Scout HQ base URL |
| `ScoutToken` | `string` | FI token issued by Scout HQ |
| `HttpClient` | `HttpClient?` | Optional caller-provided HTTP client |

The token is sent to Scout HQ as:

```http
X-Scout-Token: <fi-token>
```

## AuthorRuleBundleRequest

| Property | Type | Description |
|----------|------|-------------|
| `ProfileKey` | `string` | Authoring profile key, for example `consumer-qc.consumer-loan-qc.policy-cel-authoring` |
| `Version` | `string` | Bundle version, conventionally `YYYY.MM.DD.N` |
| `Sources` | `IReadOnlyList<AuthoringSource>` | PDFs and free-form text used for authoring |

The SDK enforces that wrapper `version` and nested `bundle_json.version` match this value.

## AuthoringSource

Represents source material supplied by the partner application.

```csharp
var pdf = AuthoringSource.FromPdf("consumer-loan-policy.pdf");
var text = AuthoringSource.FromText("checklist.txt", checklistText);
```

| Property | Description |
|----------|-------------|
| `FileName` | Display/source name |
| `MediaType` | `application/pdf` or `text/plain` |
| `Content` | Raw source bytes or text |

## AuthorRuleBundleResult

| Property | Description |
|----------|-------------|
| `EvidenceReportMarkdown` | Source-grounded evidence report |
| `RuleBundleWrapperJson` | Serialized CRM-compatible wrapper |
| `RuleBundleWrapper` | Typed wrapper object |
| `Validation` | CEL compile and field-path validation status |
| `ProfileKey` | Profile used |
| `ProfileVersion` | Profile version used |
| `InputTokens` / `OutputTokens` | Provider token usage, when available |

## RuleBundleWrapper

```csharp
public sealed class RuleBundleWrapper
{
    public string ProductKey { get; init; }
    public string WorkflowKey { get; init; }
    public string Version { get; init; }
    public JsonElement BundleJson { get; init; }
    public bool Active { get; init; }
}
```

Serialized JSON uses the CRM/HQ field names:

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

## Scout HQ Endpoint

The SDK retrieves the authoring profile through Scout HQ:

```http
GET /authoring-profiles/{profileKey}
X-Scout-Token: <fi-token>
```

Response shape:

```json
{
  "institution": {
    "id": "fi-123",
    "name": "Example Bank"
  },
  "providers": {
    "anthropic": {
      "apiKey": "..."
    }
  },
  "profile": {
    "profileKey": "consumer-qc.consumer-loan-qc.policy-cel-authoring",
    "productKey": "consumer-qc",
    "workflowKey": "consumer-loan-qc",
    "bundleProductId": "consumer-loan-qc",
    "model": "claude-sonnet-4-6",
    "maxTokens": 16384,
    "systemPrompt": "...",
    "fieldDictionary": {},
    "referenceBundle": {},
    "oracleFixture": {},
    "validationPolicy": {},
    "version": "2026.06.03.1"
  }
}
```
