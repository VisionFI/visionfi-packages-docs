# Security & Data Sovereignty

## Core Boundary

Scout HQ does not receive policy PDFs, checklist documents, free-form policy text, or generated rule-bundle results in the detached authoring flow.

Scout HQ receives only the FI token and returns:

- institution identity
- authoring profile
- provider credentials configured for that FI token

## What Leaves The Partner Environment

| Destination | Data Sent |
|-------------|-----------|
| Scout HQ | FI token only |
| Managed inference provider | Policy PDFs/text and authoring prompt |
| VisionFI CRM | Nothing |

The partner application remains responsible for storage, approval, audit records, and later Scout HQ integration of the generated bundle.

## FI Token

The FI token authenticates the institution. The SDK uses it to retrieve:

- `institution.id`
- `institution.name`
- authoring profile content
- provider credentials

For the initial version, any valid FI token can retrieve an active authoring profile. Product entitlement checks can be added later without changing the SDK output contract.

## Provider Credentials

The FI token can return provider credentials, including an Anthropic API key. The SDK uses those credentials to call managed inference directly from the partner environment.

Partners should treat provider credentials as sensitive operational secrets:

- do not log them
- do not persist them outside approved secret storage
- do not include them in support bundles or telemetry

## Document Handling

The SDK should process source files in memory where practical. If the partner application persists documents or generated bundles, that storage is governed by the partner's own controls.

## Generated Bundle Status

Generated bundles are staged output. The SDK returns:

```json
"active": false
```

That prevents authoring from being confused with approval. Promotion or activation remains a separate governance step.

## Audit Considerations

Partners should record:

- authoring profile key and version
- bundle version
- source document names
- timestamp
- validation result
- user or service principal that requested authoring

Do not log source document content, provider credentials, FI tokens, or PII.
