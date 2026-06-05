# FAQ

## General

### What does VisionFI Policy Analyzer do?

It creates staged CEL rule bundles from policy documents and free-form policy text. The output is compatible with the rule-bundle wrapper shape used by VisionFI CRM and Scout HQ.

### Is this the old policy readiness review package?

No. The old readiness-review package produced a markdown review and draft `InstitutionConfig`. This new package produces an evidence report and a staged CEL rule-bundle wrapper.

### Does the partner need VisionFI CRM?

No. The partner application owns document collection, bundle storage, review, and later Scout HQ integration.

### What is an authoring profile?

An authoring profile is a Scout HQ record for a specific product/workflow pair. It contains the authoring prompt, model settings, field dictionary, reference bundle, oracle fixture, and validation policy.

Example:

```text
consumer-qc.consumer-loan-qc.policy-cel-authoring
```

## Tokens And Access

### Can the SDK use the FI token?

Yes. The SDK uses the FI token in the `X-Scout-Token` header to retrieve the authoring profile and provider credentials from Scout HQ.

### Are product entitlements required?

Not in the initial version. Any valid FI token can retrieve an active authoring profile.

### Does Scout HQ receive the policy documents?

No. Scout HQ receives the FI token and returns configuration. Policy documents and free-form text go directly from the partner environment to managed inference.

## Output

### What does the SDK return?

The SDK returns:

- evidence report markdown
- staged rule-bundle wrapper JSON
- validation results
- authoring profile metadata
- token usage, when available

### Why is `active` false?

Generated bundles are authoring output, not approved runtime configuration. The partner decides when a bundle is approved and later integrated with Scout HQ.

### Who stores the bundle?

The partner stores it. Scout HQ integration comes later.

## Integration

### What is the NuGet package ID?

The package ID is:

```text
VisionFI.PolicyAnalyzer
```

The package file for version `0.1.0` is:

```text
VisionFI.PolicyAnalyzer.0.1.0.nupkg
```

The current public C# namespace remains:

```csharp
using VisionFI.Scout.Authoring;
```

### Can the SDK accept free-form text?

Yes. The source list can include PDFs and text inputs.

### Can it run in ASP.NET?

Yes. The client can be registered with dependency injection and called from controllers, background jobs, or internal workflow services.

### What platforms are targeted?

Windows, macOS, and Linux on-device support are assumed for the package direction.

## Troubleshooting

### The authoring profile returns 404

Confirm the profile key is correct and that the profile is active in Scout HQ seed/config data.

### The request is unauthorized

Confirm the FI token is active and sent as:

```http
X-Scout-Token: <fi-token>
```

### The bundle fails validation

The evidence report should explain blockers or ambiguous policy language. CEL validation errors usually mean the generated expression does not compile or references a field path outside the profile's field dictionary.
