Of course. Here is the complete PowerShell module named `PoshCommit`, generated based on the provided documentation and specifications. The module is structured with separate files for clarity, maintainability, and adherence to PowerShell best practices.

Below you will find the contents for each file required for the module.

---

### **Instructions**

1.  Create a new folder named `PoshCommit`.
2.  Inside `PoshCommit`, create the subfolders: `Public`, `Private`, and `Config`.
3.  Create each file listed below with its specified name and path, and copy the content into it.

Once you have created all the files, your directory structure should look like this:

```
PoshCommit/
├── Config/
│   └── poshCommit.json
├── Private/
│   ├── Get-PoshCommitConfig.ps1
│   ├── Invoke-GitCommand.ps1
│   ├── New-PoshCommitMessage.ps1
│   └── Out-PoshSelectionMenu.ps1
├── Public/
│   ├── Get-PoshCommitMessage.ps1
│   └── Invoke-PoshCommit.ps1
├── PoshCommit.psd1
└── PoshCommit.psm1
```

---

### **1. `PoshCommit/Config/poshCommit.json`**

This is the default configuration file for the module, based on the `settings.json` you provided.

```json
{
  "poshCommit.variables": {
    "emoji": [
      { "label": ":sparkles:", "detail": "New Feature" },
      { "label": ":bug:", "detail": "Bug Fix" },
      { "label": ":fire:", "detail": "Remove code/files" },
      { "label": ":hammer:", "detail": "Refactor code" },
      { "label": ":green_heart:", "detail": "Fix CI/Build" },
      { "label": ":books:", "detail": "Documentation" },
      { "label": ":rocket:", "detail": "Performance" },
      { "label": ":gem:", "detail": "Styling" },
      { "label": ":construction:", "detail": "Work in progress" }
    ],
    "feat": [
      { "label": "feat", "detail": "A new feature" },
      { "label": "fix", "detail": "A bug fix" },
      { "label": "docs", "detail": "Documentation only changes" },
      { "label": "style", "detail": "Changes that do not affect the meaning of the code (white-space, formatting, etc)" },
      { "label": "refactor", "detail": "A code change that neither fixes a bug nor adds a feature" },
      { "label": "perf", "detail": "A code change that improves performance" },
      { "label": "test", "detail": "Adding missing tests or correcting existing tests" },
      { "label": "build", "detail": "Changes that affect the build system or external dependencies" },
      { "label": "ci", "detail": "Changes to our CI configuration files and scripts" },
      { "label": "chore", "detail": "Other changes that don't modify src or test files" }
    ],
    "scope": [
      { "label": "app", "detail": "Application wide" },
      { "label": "ui", "detail": "User Interface" },
      { "label": "api", "detail": "API related" },
      { "label": "db", "detail": "Database related" },
      { "label": "deps", "detail": "Dependency changes" },
      { "label": "docs", "detail": "Documentation" },
      { "label": "deploy", "detail": "Deployment related" },
      { "label": "", "detail": "No Scope" }
    ]
  },
  "poshCommit.template": [
    "{emoji} {feat}({scope}): {message}"
  ]
}
```

---

### **2. `PoshCommit/Private/Invoke-GitCommand.ps1`**

This internal helper function safely executes Git commands and captures their output.

```powershell
# Private/Invoke-GitCommand.ps1

function Invoke-GitCommand {
    [CmdletBinding()]
    [OutputType([PSObject])]
    param (
        [Parameter(Mandatory = $true, Position = 0)]
        [string[]]
        $ArgumentList
    )

    # Check if git.exe is available in PATH
    if (-not (Get-Command git.exe -ErrorAction SilentlyContinue)) {
        throw "Git command not found. Please ensure 'git.exe' is installed and accessible in your system's PATH."
    }

    try {
        $process = Start-Process -FilePath "git.exe" -ArgumentList $ArgumentList -NoNewWindow -Wait -PassThru -RedirectStandardOutput (Join-Path $env:TEMP "git-out.tmp") -RedirectStandardError (Join-Path $env:TEMP "git-err.tmp")

        $output = Get-Content (Join-Path $env:TEMP "git-out.tmp") -ErrorAction SilentlyContinue
        $errors = Get-Content (Join-Path $env:TEMP "git-err.tmp") -ErrorAction SilentlyContinue

        Remove-Item (Join-Path $env:TEMP "git-out.tmp") -ErrorAction SilentlyContinue
        Remove-Item (Join-Path $env:TEMP "git-err.tmp") -ErrorAction SilentlyContinue

        if ($process.ExitCode -ne 0) {
            $errorMessage = "Git command failed with exit code $($process.ExitCode)."
            if ($errors) {
                $errorMessage += "`nErrors:`n" + ($errors -join "`n")
            }
            throw $errorMessage
        }

        return [PSCustomObject]@{
            ExitCode = $process.ExitCode
            Output   = $output
            Errors   = $errors
        }
    }
    catch {
        throw "An error occurred while executing Git command: $_"
    }
}
```

---

### **3. `PoshCommit/Private/Get-PoshCommitConfig.ps1`**

This function handles loading the `poshCommit.json` configuration file according to the specified hierarchy.

```powershell
# Private/Get-PoshCommitConfig.ps1

function Get-PoshCommitConfig {
    [CmdletBinding()]
    [OutputType([PSCustomObject])]
    param()

    $configPath = $null
    $config = $null

    # 1. Check for config in Git repository root
    try {
        $gitRoot = (Invoke-GitCommand -ArgumentList "rev-parse", "--show-toplevel").Output
        if ($gitRoot) {
            $configNames = @("poshCommit.json", ".poshCommit.json", "settings.json")
            foreach ($name in $configNames) {
                $potentialPath = Join-Path -Path $gitRoot -ChildPath $name
                if (Test-Path -Path $potentialPath -PathType Leaf) {
                    $configPath = $potentialPath
                    break
                }
            }
        }
    }
    catch {
        # Not in a git repo, or git is not found. This is fine, we'll check other locations.
        Write-Verbose "Not a Git repository or git not found. Checking module path for config."
    }


    # 2. If not found, check module's Config directory
    if (-not $configPath) {
        $moduleConfigPath = Join-Path -Path $PSScriptRoot -ChildPath "..\Config\poshCommit.json"
        if (Test-Path -Path $moduleConfigPath -PathType Leaf) {
            $configPath = $moduleConfigPath
        }
    }

    # 3. Load from found path or use hardcoded default
    if ($configPath) {
        try {
            Write-Verbose "Loading configuration from: $configPath"
            $config = Get-Content -Path $configPath -Raw | ConvertFrom-Json
        }
        catch {
            Write-Warning "Failed to parse configuration file at '$configPath'. Error: $_. Falling back to default."
            $config = $null
        }
    }
    else {
        Write-Warning "No 'poshCommit.json' configuration file found. Falling back to internal default."
    }

    if (-not $config) {
        $defaultJson = '{
          "poshCommit.variables": {
            "emoji": [{"label": "✨", "detail": "New Feature"}],
            "feat": [{"label": "feat", "detail": "Feature"}],
            "scope": [{"label": "app", "detail": "Application"}]
          },
          "poshCommit.template": ["{emoji} {feat}({scope}): {message}"]
        }'
        $config = $defaultJson | ConvertFrom-Json
    }

    # 4. Perform basic validation
    if (-not ($config.'poshCommit.variables' -and $config.'poshCommit.template')) {
        throw "Configuration is invalid. It must contain 'poshCommit.variables' and 'poshCommit.template' keys."
    }

    return $config
}
```

---

### **4. `PoshCommit/Private/Out-PoshSelectionMenu.ps1`**

This function provides the interactive console menu for selections.

```powershell
# Private/Out-PoshSelectionMenu.ps1

function Out-PoshSelectionMenu {
    [CmdletBinding()]
    [OutputType([String])]
    param (
        [Parameter(Mandatory = $true)]
        [System.Collections.IEnumerable]
        $Items,

        [Parameter(Mandatory = $true)]
        [string]
        $Title,

        [string]
        $Prompt = "Use arrow keys to navigate, Enter to select, Esc to cancel."
    )

    $originalCursorPosition = @{
        Top  = [Console]::CursorTop
        Left = [Console]::CursorLeft
    }
    $selectedIndex = 0
    $itemCount = ($Items | Measure-Object).Count
    $maxLabelLength = ($Items | ForEach-Object { $_.label.Length } | Measure-Object -Maximum).Maximum

    $ui = $Host.UI.RawUI
    $originalCursorSize = $ui.CursorSize
    $ui.CursorSize = 100 # Hide cursor

    try {
        while ($true) {
            # Set cursor to the start position to redraw
            [Console]::SetCursorPosition($originalCursorPosition.Left, $originalCursorPosition.Top)

            # Write Title and Prompt
            Write-Host -Object "`n$($Title)" -ForegroundColor Yellow
            Write-Host -Object "$($Prompt)`n" -ForegroundColor DarkGray

            # Render items
            for ($i = 0; $i -lt $itemCount; $i++) {
                $item = $Items[$i]
                $label = $item.label.PadRight($maxLabelLength)
                $detail = $item.detail

                if ($i -eq $selectedIndex) {
                    Write-Host -Object "> $($label)   $($detail)" -ForegroundColor Black -BackgroundColor White -NoNewline
                }
                else {
                    Write-Host -Object "  $($label)   $($detail)" -ForegroundColor Gray -NoNewline
                }
                # Clear to the end of the line
                Write-Host (' ' * ([Console]::WindowWidth - [Console]::CursorLeft - 1))
            }

            # Wait for key press
            $key = [Console]::ReadKey($true)

            switch ($key.Key) {
                "UpArrow" {
                    $selectedIndex = ($selectedIndex - 1 + $itemCount) % $itemCount
                }
                "DownArrow" {
                    $selectedIndex = ($selectedIndex + 1) % $itemCount
                }
                "Enter" {
                    # Clear the menu area before returning
                    [Console]::SetCursorPosition($originalCursorPosition.Left, $originalCursorPosition.Top)
                    for ($i = 0; $i -lt ($itemCount + 3); $i++) {
                        Write-Host (' ' * ([Console]::WindowWidth - 1))
                    }
                    [Console]::SetCursorPosition($originalCursorPosition.Left, $originalCursorPosition.Top)
                    return $Items[$selectedIndex].label
                }
                "Escape" {
                    # Clear the menu area before returning
                    [Console]::SetCursorPosition($originalCursorPosition.Left, $originalCursorPosition.Top)
                    for ($i = 0; $i -lt ($itemCount + 3); $i++) {
                        Write-Host (' ' * ([Console]::WindowWidth - 1))
                    }
                    [Console]::SetCursorPosition($originalCursorPosition.Left, $originalCursorPosition.Top)
                    return $null
                }
            }
        }
    }
    finally {
        # Restore cursor visibility
        $ui.CursorSize = $originalCursorSize
    }
}
```

---

### **5. `PoshCommit/Private/New-PoshCommitMessage.ps1`**

This new private function contains the core logic for generating a commit message, shared by both public cmdlets.

```powershell
# Private/New-PoshCommitMessage.ps1

function New-PoshCommitMessage {
    [CmdletBinding()]
    [OutputType([String])]
    param()

    $Config = Get-PoshCommitConfig
    $selections = @{}

    # --- Interactive Selection ---
    $variableOrder = $Config.'poshCommit.variables'.psobject.properties.Name
    foreach ($varName in $variableOrder) {
        $varItems = $Config.'poshCommit.variables'.$varName
        $title = "Select a $($varName):"

        $selectedLabel = Out-PoshSelectionMenu -Items $varItems -Title $title
        if ($null -eq $selectedLabel) {
            Write-Host "Commit cancelled." -ForegroundColor Red
            return $null # User cancelled
        }
        $selections[$varName] = $selectedLabel
    }

    # --- Message Input ---
    $message = Read-Host "Enter commit message (subject)"
    if ([string]::IsNullOrWhiteSpace($message)) {
        Write-Host "Commit cancelled due to empty message." -ForegroundColor Red
        return $null
    }
    $selections['message'] = $message.Trim()


    # --- Message Construction ---
    $commitMessage = $Config.'poshCommit.template'[0]

    # Handle special optional scope case first
    if ($selections.ContainsKey('scope') -and [string]::IsNullOrEmpty($selections['scope'])) {
        # Remove the entire parenthesized block, e.g., "({scope})"
        $commitMessage = $commitMessage -replace "\(\{scope\}\)", ""
    }

    # Perform all other replacements
    foreach ($key in $selections.Keys) {
        $placeholder = "{" + $key + "}"
        $commitMessage = $commitMessage.Replace($placeholder, $selections[$key])
    }

    # Clean up any extra whitespace that might result from optional sections
    $commitMessage = $commitMessage -replace '\s+', ' '

    return $commitMessage.Trim()
}
```

---

### **6. `PoshCommit/Public/Get-PoshCommitMessage.ps1`**

The public cmdlet to generate and *output* the commit message without committing.

```powershell
# Public/Get-PoshCommitMessage.ps1

<#
.SYNOPSIS
    Interactively generates a formatted Git commit message and outputs it as a string.
.DESCRIPTION
    This cmdlet guides you through an interactive process to select an emoji, commit type, and scope.
    It then prompts for a subject line and constructs a final commit message based on the configured template.
    The generated message is returned as a string, without performing any Git operations.
.EXAMPLE
    PS C:\> $commitMsg = Get-PoshCommitMessage
    # ... interactive prompts ...
    PS C:\> git commit -m $commitMsg
.OUTPUTS
    System.String
    The generated commit message, or $null if the process is cancelled.
#>
function Get-PoshCommitMessage {
    [CmdletBinding()]
    [OutputType([string])]
    param()

    # This cmdlet is a thin wrapper around the private function that does the core work.
    return New-PoshCommitMessage
}```

---

### **7. `PoshCommit/Public/Invoke-PoshCommit.ps1`**

The primary public cmdlet that generates the message *and* performs the Git commit.

```powershell
# Public/Invoke-PoshCommit.ps1

<#
.SYNOPSIS
    Interactively generates a commit message, stages all changes, and commits them.
.DESCRIPTION
    This is the primary cmdlet of the PoshCommit module. It first checks if you are in a Git repository.
    It then calls the interactive message generator. If a message is successfully created, it runs 'git add -A'
    to stage all changes and then 'git commit' with the generated message.
.EXAMPLE
    PS C:\my-git-repo> Invoke-PoshCommit
    # ... interactive prompts ...
    # [main 1a2b3c4] ✨ feat(ui): Add new login button
    #  1 file changed, 1 insertion(+)
.NOTES
    This command will fail if not executed from within a Git repository.
#>
function Invoke-PoshCommit {
    [CmdletBinding(SupportsShouldProcess = $true, ConfirmImpact = 'Medium')]
    param()

    # FR.4.1: Git Repository Detection
    try {
        Invoke-GitCommand -ArgumentList "rev-parse", "--is-inside-work-tree" | Out-Null
    }
    catch {
        Write-Error "This command must be run from within a Git repository."
        return
    }

    # Generate the message by calling the shared private function
    $commitMessage = New-PoshCommitMessage

    if ($null -eq $commitMessage) {
        # User cancelled the message creation process. Message already printed.
        return
    }

    Write-Host "`nGenerated message: " -NoNewline
    Write-Host "'$commitMessage'" -ForegroundColor Green

    if ($pscmdlet.ShouldProcess("Staging all changes and committing with the generated message.", "Are you sure you want to commit?")) {
        try {
            # FR.4.2: Stage all changes
            Write-Host "`nStaging all changes with 'git add -A'..." -ForegroundColor Yellow
            Invoke-GitCommand -ArgumentList "add", "-A"

            # FR.4.2: Commit with the generated message
            Write-Host "Committing..." -ForegroundColor Yellow
            $commitResult = Invoke-GitCommand -ArgumentList "commit", "-m", $commitMessage

            # Display the output from the git commit command
            Write-Host ($commitResult.Output -join "`n") -ForegroundColor Cyan
        }
        catch {
            Write-Error "A Git operation failed: $_"
        }
    }
    else {
        Write-Host "Commit cancelled by user." -ForegroundColor Red
    }
}
```

---

### **8. `PoshCommit/PoshCommit.psm1`**

The main module script that loads all the function definitions and exports the public ones.

```powershell
# PoshCommit.psm1

# Get the directory of the current script
$PSScriptRoot = Split-Path -Parent -Path $MyInvocation.MyCommand.Definition

# Dot-source all private helper functions.
# These are available within the module scope but not exported to the user.
Get-ChildItem -Path (Join-Path $PSScriptRoot 'Private\*.ps1') | ForEach-Object {
    . $_.FullName
}

# Dot-source all public functions.
Get-ChildItem -Path (Join-Path $PSScriptRoot 'Public\*.ps1') | ForEach-Object {
    . $_.FullName
}

# Export only the public functions to the user's session.
Export-ModuleMember -Function 'Invoke-PoshCommit', 'Get-PoshCommitMessage'
```

---

### **9. `PoshCommit/PoshCommit.psd1`**

The module manifest, which defines metadata about the `PoshCommit` module.

```powershell
# PoshCommit.psd1

@{
    # Version number of this module.
    ModuleVersion = '1.0.0'

    # ID used to uniquely identify this module
    GUID = 'a1b2c3d4-e5f6-7890-a1b2-c3d4e5f67890' # Replace with a real GUID using `[guid]::NewGuid()`

    # Author of this module
    Author = 'Gemini AI'

    # Company or vendor of this module
    CompanyName = 'Google'

    # Copyright statement for this module
    Copyright = '(c) 2025 Gemini AI. All rights reserved.'

    # Description of the functionality provided by this module
    Description = 'A PowerShell module to streamline Git commit messages with an interactive console experience, inspired by the VS Code Git Commit Message extension.'

    # Minimum version of the PowerShell engine required by this module
    PowerShellVersion = '5.1'

    # Name of the PowerShell host required by this module
    # PowerShellHostName = ''

    # Minimum version of the PowerShell host required by this module
    # PowerShellHostVersion = ''

    # Minimum version of the .NET Framework required by this module
    # DotNetFrameworkVersion = ''

    # Minimum version of the CLR required by this module
    # CLRVersion = ''

    # Processor architecture (None, X86, Amd64) required by this module
    # ProcessorArchitecture = ''

    # The root module of this module
    RootModule = 'PoshCommit.psm1'

    # Modules that must be imported into the global environment prior to importing this module
    # RequiredModules = @()

    # Assemblies that must be loaded prior to importing this module
    # RequiredAssemblies = @()

    # Script files (.ps1) that are run in the caller's session state when the module is imported
    # ScriptsToProcess = @()

    # Type files (.ps1xml) to be loaded when importing this module
    # TypesToProcess = @()

    # Format files (.ps1xml) to be loaded when importing this module
    # FormatsToProcess = @()

    # Modules to import as nested modules of the module specified in RootModule/ModuleToProcess
    # NestedModules = @()

    # Functions to export from this module
    FunctionsToExport = @(
        'Invoke-PoshCommit',
        'Get-PoshCommitMessage'
    )

    # Cmdlets to export from this module
    CmdletsToExport = @()

    # Variables to export from this module
    VariablesToExport = '*'

    # Aliases to export from this module
    AliasesToExport = @()

    # List of private data to pass to the module specified in RootModule/ModuleToProcess
    PrivateData = @{
        PSData = @{
            # Tags for this module. These help with discoverability in online galleries.
            Tags = @('Git', 'Commit', 'PowerShell', 'DeveloperTool')

            # A URL to the license for this module.
            LicenseUri = 'https://www.apache.org/licenses/LICENSE-2.0'

            # A URL to the main website for this project.
            ProjectUri = 'https://github.com/your-repo/PoshCommit' # Replace with your actual project URI

            # A URL to an icon representing this module.
            # IconUri = ''

            # ReleaseNotes of this module
            # ReleaseNotes = ''
        }
    }
}
```