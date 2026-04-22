# FAQ

## General

### What does Scout do?

Scout analyzes your institution's loan policy and checklist documents against VisionFI's consumer-loan QC workflow requirements. It produces a structured readiness assessment that identifies what's covered, what's missing, and generates a draft `InstitutionConfig` JSON that can be used to configure automated QC.

### What documents should I provide?

For the best results, provide both:

1. **Loan Policy** — the board-approved document governing consumer lending parameters
2. **Loan Checklist** — the operational document used to verify required items in the loan file

You can provide just the policy, but the review will flag the missing checklist as a gap.

### How long does a review take?

Typically 2-4 minutes, depending on document size. The time is primarily spent on the managed inference call. The SDK handles this asynchronously — your application thread is not blocked.

---

## Installation & Setup

### Do I need to install anything besides the NuGet package?

No. The NuGet package includes the native runtime for your platform. There are no separate installers, Docker containers, or services to run.

### Where does the API key come from?

VisionFI provides your Scout API key during onboarding. The SDK looks for it in this order:

1. `ScoutOptions.ApiKey` — set directly in code or app configuration
2. `SCOUT_API_KEY` environment variable

### Do I need to manage any third-party model credentials?

No. VisionFI manages the inference-layer authentication and routing. Your Scout API key is the only credential you need.

---

## Security

### Does VisionFI see my documents?

No. Scout runs entirely in your environment. Your documents are processed locally and the only external call is for managed inference, made directly from your machine.

### What is the sandbox?

The core analysis logic runs inside a secure, isolated execution environment — a sandbox. The sandboxed code has no access to your filesystem, network, or any system resources. It can only communicate with the outside world through a controlled interface that the Scout runtime manages.

### Can I run this in an air-gapped environment?

Not currently — the SDK requires network access for managed inference. If you need fully air-gapped operation, contact VisionFI to discuss on-premise deployment options.

---

## Integration

### Can I use this in an ASP.NET application?

Yes. Register the engine with dependency injection:

```csharp
builder.Services.AddScout(options =>
{
    options.ApiKey = builder.Configuration["Scout:ApiKey"];
});
```

Then inject `IScoutEngine` into your controllers or services.

### Can I cancel a long-running review?

Yes. Pass a `CancellationToken`:

```csharp
using var cts = new CancellationTokenSource(TimeSpan.FromMinutes(5));
var result = await engine.ReviewPolicyAsync(doc, cts.Token);
```

### Is the engine thread-safe?

The `ScoutEngine` is registered as a singleton and is safe to use from multiple threads. Each call to `ReviewPolicyAsync` creates an isolated execution context internally.

### What platforms are supported?

| Platform | Status |
|----------|--------|
| Windows x64 | Supported |
| macOS ARM (Apple Silicon) | Supported |
| macOS x64 | Supported |
| Linux x64 | Supported |

---

## Troubleshooting

### "No API key configured"

The SDK could not find a Scout API key. Either:

- Set `ScoutOptions.ApiKey` explicitly
- Set the `SCOUT_API_KEY` environment variable
- Contact VisionFI if you haven't received your key

### "DllNotFoundException: scout_wrapper"

The native engine library could not be found. Options:

- Ensure the NuGet package is properly installed (it includes the native binary)
- Set `ScoutOptions.NativeLibraryPath` to the library location
- Verify your platform is supported (Windows x64, macOS ARM/x64, Linux x64)

### Review takes too long or times out

Large PDF documents (10+ MB) can take several minutes to process. Use a `CancellationToken` with an appropriate timeout:

```csharp
using var cts = new CancellationTokenSource(TimeSpan.FromMinutes(10));
var result = await engine.ReviewPolicyAsync(doc, cts.Token);
```
