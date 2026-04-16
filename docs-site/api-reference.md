# API Reference

## ScoutEngine

The main entry point for policy reviews. Implements `IScoutEngine` and `IDisposable`.

### Constructors

```csharp
// Direct construction
var engine = new ScoutEngine(new ScoutOptions
{
    ApiKey = "sk-ant-...",
    NativeLibraryPath = "/optional/path/to/libscout_wrapper.dylib"
});

// With logging
var engine = new ScoutEngine(options, logger);

// Via dependency injection (resolved automatically)
public class MyService(IScoutEngine engine) { }
```

### Methods

#### `ReviewPolicyAsync`

```csharp
Task<ReviewResult> ReviewPolicyAsync(
    IEnumerable<PolicyDocument> documents,
    CancellationToken cancellationToken = default);

Task<ReviewResult> ReviewPolicyAsync(
    PolicyDocument document,
    CancellationToken cancellationToken = default);
```

Reviews one or more policy/checklist PDF documents. The operation runs asynchronously — the calling thread is not blocked.

**Throws:**

- `ScoutException` — if the review engine encounters an error
- `OperationCanceledException` — if the cancellation token is triggered
- `ArgumentException` — if no documents are provided

**Example:**

```csharp
try
{
    var result = await engine.ReviewPolicyAsync(doc);
    Console.WriteLine(result.MarkdownReport);
}
catch (ScoutException ex)
{
    Console.Error.WriteLine($"Review failed: {ex.Message}");
}
```

---

## ScoutOptions

Configuration for the Scout engine.

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `ApiKey` | `string?` | `null` | Anthropic API key. If null, resolved from `ANTHROPIC_API_KEY` env var or `~/.anthropic/api-key` file. |
| `NativeLibraryPath` | `string?` | `null` | Path to the native Scout library. If null, uses default .NET native library resolution. |

---

## PolicyDocument

Represents a PDF document to review.

### Static Factory Methods

```csharp
// From a file path
var doc = PolicyDocument.FromFile("/path/to/policy.pdf");

// From a stream (e.g. uploaded file in ASP.NET)
var doc = await PolicyDocument.FromStreamAsync(stream, "policy.pdf", cancellationToken);
```

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `FileName` | `string` | Display name for the document |
| `Content` | `ReadOnlyMemory<byte>` | Raw PDF bytes |

### Manual Construction

```csharp
var doc = new PolicyDocument
{
    FileName = "policy.pdf",
    Content = pdfBytes,
};
```

---

## ReviewResult

The result of a policy review.

| Property | Type | Description |
|----------|------|-------------|
| `MarkdownReport` | `string` | Full markdown review report |
| `InstitutionConfigJson` | `string?` | Draft config JSON, if verdict is READY or READY WITH CLARIFICATIONS |
| `InputTokens` | `int?` | Input tokens consumed |
| `OutputTokens` | `int?` | Output tokens generated |
| `Error` | `string?` | Error message (null on success) |

---

## ScoutException

Thrown when a Scout operation fails. Extends `System.Exception`.

```csharp
try
{
    var result = await engine.ReviewPolicyAsync(doc);
}
catch (ScoutException ex)
{
    // ex.Message contains the error detail
    logger.LogError(ex, "Policy review failed");
}
```

---

## IScoutEngine

Interface for the Scout engine. Use this for dependency injection and testing.

```csharp
public interface IScoutEngine : IDisposable
{
    Task<ReviewResult> ReviewPolicyAsync(
        IEnumerable<PolicyDocument> documents,
        CancellationToken cancellationToken = default);

    Task<ReviewResult> ReviewPolicyAsync(
        PolicyDocument document,
        CancellationToken cancellationToken = default);
}
```

---

## Dependency Injection

### Registration

```csharp
// With configuration
builder.Services.AddScout(options =>
{
    options.ApiKey = builder.Configuration["Scout:ApiKey"];
});

// With default options (API key from env var or file)
builder.Services.AddScout();
```

### Usage

```csharp
public class PolicyReviewController(IScoutEngine scout) : ControllerBase
{
    [HttpPost("review")]
    public async Task<IActionResult> Review(IFormFile file, CancellationToken ct)
    {
        await using var stream = file.OpenReadStream();
        var doc = await PolicyDocument.FromStreamAsync(stream, file.FileName, ct);
        var result = await scout.ReviewPolicyAsync(doc, ct);
        return Ok(result);
    }
}
```

### appsettings.json

```json
{
  "Scout": {
    "ApiKey": "sk-ant-..."
  }
}
```

The engine is registered as a **singleton** — it's safe to share across requests.
