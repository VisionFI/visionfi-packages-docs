# Getting Started

This guide shows the partner-facing flow for VisionFI Policy Analyzer.

## Prerequisites

- **.NET 9.0+**
- **Scout HQ base URL**
- **FI token** issued by Scout HQ
- Network access from the partner environment to Scout HQ and the configured managed inference provider

## Installation

Package ID:

```text
VisionFI.PolicyAnalyzer
```

Current package file:

```text
VisionFI.PolicyAnalyzer.0.1.0.nupkg
```

=== "NuGet CLI"

    ```bash
    dotnet add package VisionFI.PolicyAnalyzer --version 0.1.0
    ```

=== "PackageReference"

    ```xml
    <PackageReference Include="VisionFI.PolicyAnalyzer" Version="0.1.0" />
    ```

The package ID is `VisionFI.PolicyAnalyzer`. The current public C# namespace is `VisionFI.Scout.Authoring`.

For temporary file-based distribution, place the `.nupkg` in a local folder and add that folder as a NuGet source:

```bash
dotnet nuget add source ./nuget-local --name VisionFI-Local
dotnet add package VisionFI.PolicyAnalyzer --version 0.1.0 --source ./nuget-local
```

When hosted by the VisionFI downloads portal, the stable download ID is expected to be:

```text
nuget-policy-analyzer
```

## Author a Rule Bundle

```csharp title="Program.cs"
using VisionFI.Scout.Authoring;

var client = new ScoutAuthoringClient(new ScoutAuthoringOptions
{
    HqBaseUrl = "https://scout-hq.example.com",
    ScoutToken = Environment.GetEnvironmentVariable("SCOUT_TOKEN")
});

var request = new AuthorRuleBundleRequest
{
    ProfileKey = "consumer-qc.consumer-loan-qc.policy-cel-authoring",
    Version = "2026.06.03.1",
    Sources =
    [
        AuthoringSource.FromPdf("consumer-loan-policy.pdf"),
        AuthoringSource.FromText("checklist-notes.txt", """
        Loan checklist requires credit report, signed note, proof of income,
        and collateral documentation before booking.
        """)
    ]
};

var result = await client.AuthorRuleBundleAsync(request);

Console.WriteLine(result.EvidenceReportMarkdown);
Console.WriteLine(result.RuleBundleWrapperJson);
```

## What The SDK Does

The SDK:

1. Calls Scout HQ with the FI token.
2. Retrieves the requested authoring profile.
3. Uses the token-resolved institution ID when shaping the bundle.
4. Sends source material directly to managed inference.
5. Forces wrapper and nested bundle version consistency.
6. Validates the CEL bundle before returning it.

Scout HQ does not receive the policy PDF or free-form policy text.

## Authoring Profile

An authoring profile is specific to one product/workflow pair:

```text
consumer-qc.consumer-loan-qc.policy-cel-authoring
```

The profile provides:

| Field | Purpose |
|-------|---------|
| `productKey` | Wrapper `product_key` |
| `workflowKey` | Wrapper `workflow_key` |
| `bundleProductId` | Nested `bundle_json.product_id` |
| `systemPrompt` | Authoring instructions |
| `fieldDictionary` | Valid `pkg.*` paths |
| `referenceBundle` | Known-good bundle shape |
| `oracleFixture` | Validation fixture for path linting |
| `validationPolicy` | Compile/path-lint requirements |

## Output

`AuthorRuleBundleResult` contains:

| Property | Description |
|----------|-------------|
| `EvidenceReportMarkdown` | Source-grounded explanation of what can and cannot become deterministic rules |
| `RuleBundleWrapperJson` | CRM/HQ-compatible staged rule-bundle wrapper |
| `Validation` | Local validation result, including CEL compile and field-path checks |
| `ProfileVersion` | Authoring profile version used |
| `InputTokens` / `OutputTokens` | Token usage, when returned by the provider |

## Storage And Approval

The partner owns storage, review, approval, and later Scout HQ integration. The SDK returns `active: false` because generated bundles are staged authoring output, not automatically approved runtime configuration.
