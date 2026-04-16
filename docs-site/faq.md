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

Typically 2-4 minutes, depending on document size. The time is primarily spent on the LLM inference call. The SDK handles this asynchronously — your application thread is not blocked.

### What LLM model is used?

Scout uses Claude Opus 4.6 via the Anthropic Messages API for maximum accuracy on complex document analysis.

---

## Installation & Setup

### Do I need to install anything besides the NuGet package?

No. The NuGet package includes the native runtime for your platform. There are no separate installers, Docker containers, or services to run.

### Where does the API key come from?

The SDK looks for an Anthropic API key in this order:

1. `ScoutOptions.ApiKey` — set directly in code or configuration
2. `ANTHROPIC_API_KEY` environment variable
3. `~/.anthropic/api-key` file

### Can I use my organization's Anthropic enterprise account?

Yes. You provide your own API key. Scout makes calls directly to the Anthropic API from your environment. Any enterprise agreements, rate limits, or data policies you've negotiated with Anthropic apply automatically.

---

## Security

### Does VisionFI see my documents?

No. Scout runs entirely in your environment. Your documents are processed locally and sent directly to the Anthropic API by code running on your machine. VisionFI has no access to your data.

### What is WebAssembly containment?

The core analysis logic runs inside a WebAssembly (Wasm) sandbox — a secure, isolated execution environment. The sandboxed code has no access to your filesystem, network, or any system resources. It can only communicate with the outside world through explicitly defined host functions that the Scout runtime controls.

### Can I run this in an air-gapped environment?

Not currently — the SDK requires network access to reach the Anthropic API for LLM inference. If you need fully air-gapped operation, contact VisionFI to discuss on-premise LLM deployment options.

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

The `ScoutEngine` is registered as a singleton and is safe to use from multiple threads. Each call to `ReviewPolicyAsync` creates an isolated Wasm instance internally.

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

The SDK could not find an Anthropic API key. Either:

- Set `ScoutOptions.ApiKey` explicitly
- Set the `ANTHROPIC_API_KEY` environment variable
- Create a file at `~/.anthropic/api-key` containing your key

### "DllNotFoundException: scout_wrapper"

The native library could not be found. Options:

- Ensure the NuGet package is properly installed (it includes the native binary)
- Set `ScoutOptions.NativeLibraryPath` to the library location
- On macOS/Linux, set `DYLD_LIBRARY_PATH` or `LD_LIBRARY_PATH` to include the directory

### Review takes too long or times out

Large PDF documents (10+ MB) combined with Claude's analysis can take several minutes. Use a `CancellationToken` with an appropriate timeout:

```csharp
using var cts = new CancellationTokenSource(TimeSpan.FromMinutes(10));
var result = await engine.ReviewPolicyAsync(doc, cts.Token);
```
