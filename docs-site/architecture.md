# Architecture

## Detached Authoring Model

The Scout Authoring SDK is designed for partners who need to create rule bundles without using VisionFI CRM.

```text
Partner Application
  -> Scout Authoring SDK
      -> Scout HQ
          - FI token validation
          - institution identity
          - authoring profile
          - provider credentials
      -> Managed inference
          - policy PDFs/text
          - profile prompt and schema artifacts
      -> Local validation
      -> CRM-compatible rule-bundle wrapper
```

Scout HQ is the control plane for identity and authoring configuration. It is not the document processor in this flow.

## Scout HQ Responsibilities

Scout HQ provides:

- FI token validation
- Institution ID and institution name
- Product/workflow-specific authoring profiles
- Provider credentials associated with the FI token
- Versioned schema artifacts used by the SDK

The same FI token used by Scout runtime flows can retrieve authoring profiles. For the initial partner flow, no product entitlement check is required; a valid token is sufficient.

## SDK Responsibilities

The SDK handles:

- PDF and text source preparation
- Authoring request construction
- Managed inference calls from the partner environment
- Evidence report parsing
- Rule-bundle wrapper shaping
- CEL bundle validation
- Version consistency checks

The SDK uses the token-resolved `institution.id` from Scout HQ. The caller does not supply the institution ID.

## Authoring Profile Boundary

Authoring profiles are distinct from Scout Chat agents and Scout runtime workflows.

| HQ object | Purpose |
|-----------|---------|
| `agents` | Scout Chat directives |
| `workflows` | Scout runtime workflow configuration |
| `rule_bundles` | Institution-specific executable CEL bundles |
| `authoring_profiles` | Instructions and schema artifacts for creating CEL bundles |

This keeps the authoring package from changing Scout Chat behavior or Scout runtime execution.

## Data Flow

```text
FI token
  -> Scout HQ
  -> institution identity + authoring profile + provider credentials

Policy PDFs / free-form text
  -> SDK
  -> managed inference provider
  -> SDK
  -> partner-owned storage
```

## Output Contract

The SDK returns the CRM-compatible wrapper:

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

`active` remains `false` because promotion is a separate governance action.

## Platform Support

The target package shape supports partner-hosted .NET applications on:

| Platform | Status |
|----------|--------|
| Windows x64 | Planned |
| macOS ARM | Planned |
| macOS x64 | Planned |
| Linux x64 | Planned |
