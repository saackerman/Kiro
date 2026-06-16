# Instructions add to .kiro/steering/powershell-api.md
# PowerShell 7.5+ REST API Coding Standards

## Core Cmdlet Requirements
* Always use `Invoke-RestMethod` (or alias `irm`) for interacting with REST endpoints.
* Leverage modern parameters like `-StatusCodeVariable` and `-ResponseHeadersVariable` for robust response tracking.
* Do not pipe `Invoke-RestMethod` output to another command without explicitly handling array boundaries (`[Object[]]`).

## Mandatory Splatting Pattern
* **Rule:** Always use splatting for API calls. Consolidate your arguments into an array or a hash table named `$Params`, and pass it using `@Params`. Do not string long lists of parameters inline.
* **Reason:** This keeps code scannable, clean, and modular.

## Payload Handling & JSON Depth
* Explicitly pass `-ContentType "application/json"` on all mutation operations (`POST`, `PUT`, `PATCH`).
* **Rule:** Always append `-Depth 10` (or higher if required) when using `ConvertTo-Json`. 
* **Reason:** PowerShell defaults to a depth limit of 2. Forgetting this truncates nested objects into simple strings, breaking SaaS payloads.
* Use Ordered Hashtables (`[ordered]@{}`) before converting to JSON to ensure predictable payload structures.

## Error Handling
* Always pass `-SkipHttpErrorCheck` if you need to manually evaluate and log specific HTTP error codes via the `-StatusCodeVariable`.

## Examples
* **Good:**
  ```powershell
  \$Payload = [ordered]@{
      name = "azure-vm-01"
      tags = @{ env = "production"; tier = "web" }
  } | ConvertTo-Json -Depth 10

  \$Params = @{
      Uri                = "https://azure.com"
      Method             = "Post"
      Body               = \$Payload
      ContentType        = "application/json"
      StatusCodeVariable = "HttpCode"
  }
  \$Response = Invoke-RestMethod @Params
  ```
* **Bad:**
  ```powershell
  # Inline parameter bloat, no ordered hash, and default depth will truncate the 'tags' object
  \$Body = @{ name = "azure-vm-01"; tags = @{ env = "production" } } | ConvertTo-Json
  Invoke-RestMethod -Uri "https://..." -Method Post -Body \$Body -ContentType "application/json"
  ```
