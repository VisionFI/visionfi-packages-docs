# Getting Started

Get a policy review running in under 5 minutes.

## Prerequisites

- **.NET 9.0+** — [Download](https://dotnet.microsoft.com/download)
- **Scout API key** — provided by VisionFI during onboarding

## Installation

=== "NuGet CLI"

    ```bash
    dotnet add package VisionFI.Scout
    ```

=== "PackageReference"

    ```xml
    <PackageReference Include="VisionFI.Scout" Version="0.1.0" />
    ```

The NuGet package includes the native runtime for your platform — nothing else to install.

## Your First Review

### 1. Create a console app

```bash
dotnet new console -n PolicyReview
cd PolicyReview
dotnet add package VisionFI.Scout
```

### 2. Write the code

```csharp title="Program.cs"
using VisionFI.Scout;

// Create the engine with the key VisionFI provided
var engine = new ScoutEngine(new ScoutOptions
{
    ApiKey = "your-visionfi-scout-key"
});

// Load your policy PDF
var doc = PolicyDocument.FromFile("consumer-loan-policy.pdf");

// Run the review
Console.WriteLine("Reviewing policy...");
var result = await engine.ReviewPolicyAsync(doc);

// Output
Console.WriteLine(result.MarkdownReport);
Console.WriteLine($"\nTokens: in={result.InputTokens}  out={result.OutputTokens}");
```

### 3. Run it

```bash
dotnet run
```

!!! tip "Configuration alternatives"
    Instead of hardcoding the key, you can set it via environment variable (`SCOUT_API_KEY`) or load it from your app's configuration:
    ```csharp
    var engine = new ScoutEngine(new ScoutOptions
    {
        ApiKey = Configuration["Scout:ApiKey"]
    });
    ```

## Multiple Documents

Pass both a loan policy and checklist for the most complete review:

```csharp
var documents = new[]
{
    PolicyDocument.FromFile("consumer-loan-policy.pdf"),
    PolicyDocument.FromFile("loan-checklist.pdf"),
};

var result = await engine.ReviewPolicyAsync(documents);
```

## What You Get Back

The `ReviewResult` contains:

| Property | Type | Description |
|----------|------|-------------|
| `MarkdownReport` | `string` | Full review with readiness verdict, coverage matrix, blockers, and risk analysis |
| `InstitutionConfigJson` | `string?` | Draft QC configuration JSON (if verdict is READY or READY WITH CLARIFICATIONS) |
| `InputTokens` | `int?` | Input token count |
| `OutputTokens` | `int?` | Output token count |

The markdown report includes:

1. **Overall readiness verdict** — READY, READY WITH CLARIFICATIONS, or INSUFFICIENT
2. **Coverage matrix** — status of all 9 QC dimensions with quoted evidence
3. **Blockers** — concrete questions for the bank, phrased as config values
4. **Soft gaps** — nice-to-have clarifications
5. **Risk analysis** — contradictions, stale content, vague language
6. **Draft InstitutionConfig** — JSON ready to plug into VisionFI's QC workflow

## Next Steps

- [API Reference](api-reference.md) — full type documentation
- [Architecture](architecture.md) — how the containment model works
- [Security](security.md) — data sovereignty and IP protection details
