# Instructions .kiro/steering/cloud-auth.md
# Azure, AWS & SaaS Security/Authentication Standards

## Token & Header Isolation
* Never hardcode API Keys, Bearer tokens, or Secret keys into script blocks.
* Consolidate headers into a reusable `$Headers` hashtable using splatting patterns.

## Platform Specific Constraints
* **Azure:** Build authorization blocks utilizing the `Bearer $Token` convention pulled dynamically from `Get-AzAccessToken` or Managed Identities.
* **AWS:** Utilize signature version 4 structures or default to the AWSPowerShell SDK configuration namespaces if raw REST endpoints are bypassed.
* **SaaS Tools:** Enforce the usage of explicit `-Authentication Bearer` or `CustomMethod` parameters natively available in PS 7.5+.

## Output Preservation
* Silence verbose network streams by default unless `-Verbose` parameter block is explicitly requested by the parent module.

## Examples
* **Good:**
  ```powershell
  \$Headers = @{
      "Authorization" = "Bearer \$AzureToken"
      "x-ms-version"  = "2023-11-01"
  }
  Invoke-RestMethod -Uri AzureEndpoint -Headers Headers -Method Get
  ```
