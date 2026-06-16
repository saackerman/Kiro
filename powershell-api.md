# Instructions add to .kiro/steering/powershell-api.md

# PowerShell 7.5+ REST API Coding Standards

## Core Cmdlet Requirements
* Always use `Invoke-RestMethod` (or alias `irm`) for interacting with REST endpoints.
* Leverage modern parameters like `-StatusCodeVariable` and `-ResponseHeadersVariable` for robust response tracking instead of try/catch parsing.
* Do not pipe `Invoke-RestMethod` output to another command without explicitly handling array boundaries (`[Object[]]`).

## Payload Handling & JSON Formatting
* Explicitly pass `-ContentType "application/json"` on all mutation operations (`POST`, `PUT`, `PATCH`).
* When generating JSON payloads, use `ConvertTo-Json -Depth 10` to avoid implicit truncation of nested arrays/objects.
* Use Ordered Hashtables (`[ordered]@{}`) before converting to JSON to ensure predictable payload structures.

## Error Handling
* Always pass `-SkipHttpErrorCheck` if you need to manually evaluate and log specific HTTP error codes via the `-StatusCodeVariable`.

## Examples
* **Good:**
  ```powershell
  \$Payload = [ordered]@{
      name = "azure-vm-01"
      tags = @{ env = "production" }
  } | ConvertTo-Json -Depth 5

  \$Params = @{
      Uri                = "https://azure.com"
      Method             = "Post"
      Body               = \$Payload
      ContentType        = "application/json"
      StatusCodeVariable = "HttpCode"
  }
  \$Response = Invoke-RestMethod @Params
  ```
* **Bad (Legacy / Obsolete):**
  ```powershell
  # Missing explicit depth, missing modern status code checks, no ordered hashtable
  \$Body = @{ name = "azure-vm-01"; tags = @{ env = "production" } } | ConvertTo-Json
  try { Invoke-RestMethod -Uri \$Uri -Method Post -Body Body catch _.Exception }
  ```
