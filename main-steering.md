# Instructions
.kiro/steering/main-steering.md
# PowerShell 7.5+ Unified Development & Testing Standards

## 1. Core Principles & General Splatting Rules
* **Rule:** Use splatting for any cmdlet, advanced function, or script call that uses **three (3) or more parameters**, or when any single parameter line exceeds **80 characters**.
* **Naming Convention:** Name the splatting variable relative to its target (e.g., `$Params`, `$AzParams`, `$SaaSParams`, `$MailParams`).
* **Format:** Declare parameters line-by-line within an ordered block `[ordered]@{}` or a standard block `@{}` with a single space surrounding the `=` sign. Call the target using `@` (e.g., `Cmdlet-Name @Params`).
* **Downstream Forwarding:** Clean dynamic variables from `$PSBoundParameters` using `.Remove('ParamName')` before splatting it downstream to prevent parameter binding errors.

---

## 2. Error Handling & Stream Architectures
* **Rule:** Prefer native, modern parameters like `-StatusCodeVariable`, `-ErrorVariable`, or explicit `-ErrorAction Stop` over broad, generic `try/catch` blocks wherever possible.
* **Stream Tracking:** Use native PowerShell 7.5+ variables to isolate stream outputs instead of piping into complex filtering loops.
* **Exit Controls:** Advanced functions must utilize `return` blocks or throw terminal exceptions explicitly using `Throw` rather than using script-ending `Exit` commands, allowing outer orchestration engines to catch the failure.

---

## 3. REST API & JSON Standards
* **Rule:** Always use `Invoke-RestMethod` (alias `irm`) for REST interactions. 
* **JSON Depth Protection:** Always explicitly append `-Depth 10` (or higher) to all instances of `ConvertTo-Json`. Do not rely on the PowerShell default depth of 2, which truncates nested SaaS payloads.
* **Payload Format:** Force the use of `[ordered]@{}` prior to converting objects to JSON to guarantee predictable structural parsing on SaaS servers.
* **Error Overrides:** Use `-SkipHttpErrorCheck` when you need to manually evaluate bad payloads or non-200 responses captured via `-StatusCodeVariable`.

---

## 4. Azure, AWS & SaaS Security Standards
* **Credential Isolation:** Never allow hardcoded API keys, bearer tokens, or secrets inside the code block. Pass them dynamically via secure parameters or fetch them at runtime.
* **Azure Conventions:** Build authentication headers using `Bearer $Token` pulled natively from system-assigned identities or `Get-AzAccessToken`.
* **SaaS Integration:** Utilize the native `-Authentication` parameter blocks supported dynamically in PowerShell 7+.

---

## 5. Pester 5+ Unit Testing Architecture
* **Phase Separation:** Tests must decouple discovery from execution. Never put setup logic, variable initialization, or mock definitions directly into `Describe` or `Context` blocks unless wrapped inside a `BeforeAll` block.
* **Endpoint Isolation:** Always mock network-mutating cmdlets (`Invoke-RestMethod`, `Invoke-WebRequest`) using `-ParameterFilter` blocks to prevent live mutations of cloud resources during test cycles.
* **Assertions:** Only use modern semantic assertions (`Should -Be`, `Should -Not -BeNullOrEmpty`, `Should -Throw`). Legacy Pester 4 `Be` syntax is forbidden.

---

## 6. Comprehensive Examples

### Good (Advanced Function with Clean Splatting, JSON Processing & Error Action Handling):
```powershell
function Invoke-SaaSDeployment {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)] [string]\$EndpointUrl,
        [Parameter(Mandatory)] [hashtable]\$PayloadData
    )

    # Prevent depth truncation natively and enforce ordered tracking
    \$JsonBody = [ordered]@{
        requestTimestamp = (Get-Date -Format 'o')
        data             = \$PayloadData
    } | ConvertTo-Json -Depth 10

    # Modern parameter isolation with strict error actions using splatting
    \$SaaSParams = @{
        Uri                = \$EndpointUrl
        Method             = 'Post'
        Body               = \$JsonBody
        ContentType        = 'application/json'
        ErrorAction        = 'Stop'
        StatusCodeVariable = 'HttpResponseCode'
    }

    Invoke-RestMethod @SaaSParams
}
```

### Good (Pester 5+ Lifecycle and API Mocking):
```powershell
Describe "Invoke-SaaSDeployment Test Suite" {
    BeforeAll {
        \$MockedOutput = [PSCustomObject]@{ status = 'Success'; jobId = '999' }
    }

    Context "When sending a valid network configuration payload" {
        BeforeAll {
            Mock Invoke-RestMethod { return \$MockedOutput } -ParameterFilter {
                Uri -like "*api.saas.local*" -and Method -eq "Post"
            }
        }

        It "Should process the mock result correctly without throwing error streams" {
            \$Result = Invoke-SaaSDeployment -EndpointUrl "https://saas.local" -PayloadData @{ env = 'dev' }
            \(Result.status \vert{} Should -Be 'Success'\)Result.jobId  | Should -Be '999'
        }
    }
}
```
