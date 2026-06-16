# Instructions
.kiro/steering/pester-tests.md

# Pester 5+ Unit Testing Standards

## Framework Architecture
* Enforce Pester 5+ syntax. Tests must decouple the discovery phase from the execution phase.
* Never put setup logic, variables, or mock code directly inside `Describe` or `Context` blocks unless enclosed in a `BeforeAll` block.

## Mocking REST APIs
* Always mock `Invoke-RestMethod` and `Invoke-WebRequest` to prevent accidental live cloud/SaaS infrastructure mutations.
* Customize mock behavior using the `-ParameterFilter` switch to match specific SaaS endpoints and payloads.
* Return mock objects that mimic actual API schemas using `ConvertFrom-Json` strings.

## Assertions
* Use modern Pester assertions (`Should -Be`, `Should -Not -BeNullOrEmpty`, `Should -Throw`).

## Examples
* **Good:**
  ```powershell
  BeforeAll {
      # Setup reusable test tokens or paths
  }

  Describe "Get-AzureSaaSTag" {
      BeforeAll {
          # Define mock response object mimicking real SaaS structure
          \$MockResponse = [PSCustomObject]@{ id = "123"; status = "Active" }
          
          # Mock the API call cleanly
          Mock Invoke-RestMethod { return \$MockResponse } -ParameterFilter { 
              Uri -like "*://azure.com*" -and Method -eq "Get" 
          }
      }

      Context "When the API returns a valid object" {
          It "Should successfully output the profile state as Active" {
              \$Result = Get-AzureSaaSTag -ResourceId "123"
              \$Result.status | Should -Be "Active"
          }
      }
  }
  ```
* **Bad (Legacy Pester 4 Syntax / Broken Lifecycle):**
  ```powershell
  Describe "Legacy Test Structure" {
      # WRONG: Setting variables directly in the block triggers Discovery errors in Pester 5
      \$MockResponse = @{ status = "Active" } 
      Mock Invoke-RestMethod { \$MockResponse }
      
      It "Should test status" {
          (Get-AzureSaaSTag).status | Should Be "Active" # Obsolete 'Be' syntax
      }
  }
  ```
