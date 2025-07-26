Of course. Here is a PowerShell script that will automatically create the entire `PoshCommit` module directory structure and populate it with the correct files and content.

### Instructions

1.  Open a PowerShell terminal.
2.  Navigate to the directory where you want to create the `PoshCommit` module folder (for example, your desktop or a projects folder).
3.  Copy the entire script below and paste it into your PowerShell terminal.
4.  Press `Enter` to run the script.

The script will create a `PoshCommit` folder in your current directory, complete with all the necessary subfolders and files. After it finishes, it will provide instructions on how to install and use the module.

---

### **`Create-PoshCommitModule.ps1`**

```powershell
#
# Create-PoshCommitModule.ps1
#
# This script generates the complete folder structure and all necessary files
# for the PoshCommit PowerShell module in the current directory.
#

# --- Script Start ---
Write-Host "Starting the creation of the PoshCommit module..." -ForegroundColor Yellow

# --- 1. Define Base Path and Create Directory Structure ---
$baseDir = Join-Path -Path $PSScriptRoot -ChildPath 'PoshCommit'
if (Test-Path $baseDir) {
    Write-Warning "Directory '$baseDir' already exists. To avoid data loss, the script will stop."
    Write-Warning "Please remove or rename the existing 'PoshCommit' directory and run the script again."
    return
}

Write-Host "Creating module directory structure at '$baseDir'..."
$publicDir = Join-Path -Path $baseDir -ChildPath 'Public'
$privateDir = Join-Path -Path $baseDir -ChildPath 'Private'
$configDir = Join-Path -Path $baseDir -ChildPath 'Config'

New-Item -Path $baseDir -ItemType Directory | Out-Null
New-Item -Path $publicDir -ItemType Directory | Out-Null
New-Item -Path $privateDir -ItemType Directory | Out-Null
New-Item -Path $configDir -ItemType Directory | Out-Null
Write-Host "Directory structure created successfully." -ForegroundColor Green


# --- 2. Define File Contents Using PowerShell Here-Strings ---

# File: PoshCommit.psd1 (Module Manifest)
$psd1Content = @"
# PoshCommit.psd1

@{
    # Version number of this module.
    ModuleVersion = '1.0.0'

    # ID used to uniquely identify this module
    GUID = '$(New-Guid)'

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

    # The root module of this module
    RootModule = 'PoshCommit.psm1'

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
            Tags = @('Git', 'Commit', 'PowerShell', 'DeveloperTool')
            LicenseUri = 'https://www.apache.org/licenses/LICENSE-2.0'
            ProjectUri = 'https://github.com/your-repo/PoshCommit' # Replace with your actual project URI
        }
    }
}
"@

# File: PoshCommit.psm1 (Main Module Script)
$psm1Content = @"
# PoshCommit.psm1

# Get the directory of the current script
\$PSScriptRoot = Split-Path -Parent -Path \$MyInvocation.MyCommand.Definition

# Dot-source all private helper functions.
Get-ChildItem -Path (Join-Path \$PSScriptRoot 'Private\*.ps1') | ForEach-Object {
    . \`$_.FullName
}

# Dot-source all public functions.
Get-ChildItem -Path (Join-Path \$PSScriptRoot 'Public\*.ps1') | ForEach-Object {
    . \`$_.FullName
}

# Export only the public functions to the user's session.
Export-ModuleMember -Function 'Invoke-PoshCommit', 'Get-PoshCommitMessage'
"@

# File: Config/poshCommit.json (Default Configuration)
$jsonContent = @"
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
"@

# File: Public/Invoke-PoshCommit.ps1
$invokePoshCommitContent = @"
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
    [CmdletBinding(SupportsShouldProcess = \$true, ConfirmImpact = 'Medium')]
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
    \$commitMessage = New-PoshCommitMessage

    if (\$null -eq \$commitMessage) {
        # User cancelled the message creation process. Message already printed.
        return
    }

    Write-Host "`nGenerated message: " -NoNewline
    Write-Host "'\$commitMessage'" -ForegroundColor Green

    if (\$pscmdlet.ShouldProcess("Staging all changes and committing with the generated message.", "Are you sure you want to commit?")) {
        try {
            # FR.4.2: Stage all changes
            Write-Host "`nStaging all changes with 'git add -A'..." -ForegroundColor Yellow
            Invoke-GitCommand -ArgumentList "add", "-A"

            # FR.4.2: Commit with the generated message
            Write-Host "Committing..." -ForegroundColor Yellow
            \$commitResult = Invoke-GitCommand -ArgumentList "commit", "-m", "\$commitMessage"

            # Display the output from the git commit command
            Write-Host (\$commitResult.Output -join "`n") -ForegroundColor Cyan
        }
        catch {
            Write-Error "A Git operation failed: \`$_"
        }
    }
    else {
        Write-Host "Commit cancelled by user." -ForegroundColor Red
    }
}
"@

# File: Public/Get-PoshCommitMessage.ps1
$getPoshCommitMessageContent = @"
# Public/Get-PoshCommitMessage.ps1

<#
.SYNOPSIS
    Interactively generates a formatted Git commit message and outputs it as a string.
.DESCRIPTION
    This cmdlet guides you through an interactive process to select an emoji, commit type, and scope.
    It then prompts for a subject line and constructs a final commit message based on the configured template.
    The generated message is returned as a string, without performing any Git operations.
.EXAMPLE
    PS C:\> \$commitMsg = Get-PoshCommitMessage
    # ... interactive prompts ...
    PS C:\> git commit -m \$commitMsg
.OUTPUTS
    System.String
    The generated commit message, or \$null if the process is cancelled.
#>
function Get-PoshCommitMessage {
    [CmdletBinding()]
    [OutputType([string])]
    param()

    # This cmdlet is a thin wrapper around the private function that does the core work.
    return New-PoshCommitMessage
}
"@

# File: Private/Get-PoshCommitConfig.ps1
$getPoshCommitConfigContent = @"
# Private/Get-PoshCommitConfig.ps1

function Get-PoshCommitConfig {
    [CmdletBinding()]
    [OutputType([PSCustomObject])]
    param()

    \$configPath = \$null
    \$config = \$null

    # 1. Check for config in Git repository root
    try {
        \$gitRoot = (Invoke-GitCommand -ArgumentList "rev-parse", "--show-toplevel").Output
        if (\$gitRoot) {
            \$configNames = @("poshCommit.json", ".poshCommit.json", "settings.json")
            foreach (\$name in \$configNames) {
                \$potentialPath = Join-Path -Path \$gitRoot -ChildPath \$name
                if (Test-Path -Path \$potentialPath -PathType Leaf) {
                    \$configPath = \$potentialPath
                    break
                }
            }
        }
    }
    catch {
        Write-Verbose "Not a Git repository or git not found. Checking module path for config."
    }

    # 2. If not found, check module's Config directory
    if (-not \$configPath) {
        \$moduleConfigPath = Join-Path -Path \$PSScriptRoot -ChildPath "..\Config\poshCommit.json"
        if (Test-Path -Path \$moduleConfigPath -PathType Leaf) {
            \$configPath = \$moduleConfigPath
        }
    }

    # 3. Load from found path or use hardcoded default
    if (\$configPath) {
        try {
            Write-Verbose "Loading configuration from: \$configPath"
            \$config = Get-Content -Path \$configPath -Raw | ConvertFrom-Json
        }
        catch {
            Write-Warning "Failed to parse configuration file at '\$configPath'. Error: \`$_. Falling back to default."
            \$config = \$null
        }
    }
    else {
        Write-Warning "No 'poshCommit.json' configuration file found. Falling back to internal default."
    }

    if (-not \$config) {
        \$defaultJson = '{
          "poshCommit.variables": {
            "emoji": [{"label": "✨", "detail": "New Feature"}],
            "feat": [{"label": "feat", "detail": "Feature"}],
            "scope": [{"label": "app", "detail": "Application"}]
          },
          "poshCommit.template": ["{emoji} {feat}({scope}): {message}"]
        }'
        \$config = \$defaultJson | ConvertFrom-Json
    }

    # 4. Perform basic validation
    if (-not (\$config.'poshCommit.variables' -and \$config.'poshCommit.template')) {
        throw "Configuration is invalid. It must contain 'poshCommit.variables' and 'poshCommit.template' keys."
    }

    return \$config
}
"@

# File: Private/Out-PoshSelectionMenu.ps1
$outPoshSelectionMenuContent = @"
# Private/Out-PoshSelectionMenu.ps1

function Out-PoshSelectionMenu {
    [CmdletBinding()]
    [OutputType([String])]
    param (
        [Parameter(Mandatory = \$true)]
        [System.Collections.IEnumerable]
        \$Items,

        [Parameter(Mandatory = \$true)]
        [string]
        \$Title,

        [string]
        \$Prompt = "Use arrow keys to navigate, Enter to select, Esc to cancel."
    )

    \$originalCursorPosition = @{
        Top  = [Console]::CursorTop
        Left = [Console]::CursorLeft
    }
    \$selectedIndex = 0
    \$itemCount = (\$Items | Measure-Object).Count
    \$maxLabelLength = (\$Items | ForEach-Object { \`$_.label.Length } | Measure-Object -Maximum).Maximum

    \$ui = \$Host.UI.RawUI
    \$originalCursorSize = \$ui.CursorSize
    \$ui.CursorSize = 100 # Hide cursor

    try {
        while (\$true) {
            [Console]::SetCursorPosition(\$originalCursorPosition.Left, \$originalCursorPosition.Top)

            Write-Host -Object "`n\$(\$Title)" -ForegroundColor Yellow
            Write-Host -Object "\$Prompt`n" -ForegroundColor DarkGray

            for (\$i = 0; \$i -lt \$itemCount; \$i++) {
                \$item = \$Items[\$i]
                \$label = \$item.label.PadRight(\$maxLabelLength)
                \$detail = \$item.detail

                if (\$i -eq \$selectedIndex) {
                    Write-Host -Object "> \$(\$label)   \$(\$detail)" -ForegroundColor Black -BackgroundColor White -NoNewline
                }
                else {
                    Write-Host -Object "  \$(\$label)   \$(\$detail)" -ForegroundColor Gray -NoNewline
                }
                Write-Host (' ' * ([Console]::WindowWidth - [Console]::CursorLeft - 1))
            }

            \$key = [Console]::ReadKey(\$true)

            switch (\$key.Key) {
                "UpArrow" { \$selectedIndex = (\$selectedIndex - 1 + \$itemCount) % \$itemCount }
                "DownArrow" { \$selectedIndex = (\$selectedIndex + 1) % \$itemCount }
                "Enter" {
                    [Console]::SetCursorPosition(\$originalCursorPosition.Left, \$originalCursorPosition.Top)
                    for (\$i = 0; \$i -lt (\$itemCount + 3); \$i++) { Write-Host (' ' * ([Console]::WindowWidth - 1)) }
                    [Console]::SetCursorPosition(\$originalCursorPosition.Left, \$originalCursorPosition.Top)
                    return \$Items[\$selectedIndex].label
                }
                "Escape" {
                    [Console]::SetCursorPosition(\$originalCursorPosition.Left, \$originalCursorPosition.Top)
                    for (\$i = 0; \$i -lt (\$itemCount + 3); \$i++) { Write-Host (' ' * ([Console]::WindowWidth - 1)) }
                    [Console]::SetCursorPosition(\$originalCursorPosition.Left, \$originalCursorPosition.Top)
                    return \$null
                }
            }
        }
    }
    finally {
        \$ui.CursorSize = \$originalCursorSize
    }
}
"@

# File: Private/Invoke-GitCommand.ps1
$invokeGitCommandContent = @"
# Private/Invoke-GitCommand.ps1

function Invoke-GitCommand {
    [CmdletBinding()]
    [OutputType([PSObject])]
    param (
        [Parameter(Mandatory = \$true, Position = 0)]
        [string[]]
        \$ArgumentList
    )

    if (-not (Get-Command git.exe -ErrorAction SilentlyContinue)) {
        throw "Git command not found. Please ensure 'git.exe' is installed and accessible in your system's PATH."
    }

    try {
        \$process = Start-Process -FilePath "git.exe" -ArgumentList \$ArgumentList -NoNewWindow -Wait -PassThru -RedirectStandardOutput (Join-Path \$env:TEMP "git-out.tmp") -RedirectStandardError (Join-Path \$env:TEMP "git-err.tmp")

        \$output = Get-Content (Join-Path \$env:TEMP "git-out.tmp") -ErrorAction SilentlyContinue
        \$errors = Get-Content (Join-Path \$env:TEMP "git-err.tmp") -ErrorAction SilentlyContinue

        Remove-Item (Join-Path \$env:TEMP "git-out.tmp") -ErrorAction SilentlyContinue
        Remove-Item (Join-Path \$env:TEMP "git-err.tmp") -ErrorAction SilentlyContinue

        if (\$process.ExitCode -ne 0) {
            \$errorMessage = "Git command failed with exit code \$(\$process.ExitCode)."
            if (\$errors) {
                \$errorMessage += "`nErrors:`n" + (\$errors -join "`n")
            }
            throw \$errorMessage
        }

        return [PSCustomObject]@{
            ExitCode = \$process.ExitCode
            Output   = \$output
            Errors   = \$errors
        }
    }
    catch {
        throw "An error occurred while executing Git command: \`$_"
    }
}
"@

# File: Private/New-PoshCommitMessage.ps1
$newPoshCommitMessageContent = @"
# Private/New-PoshCommitMessage.ps1

function New-PoshCommitMessage {
    [CmdletBinding()]
    [OutputType([String])]
    param()

    \$Config = Get-PoshCommitConfig
    \$selections = @{}

    # --- Interactive Selection ---
    \$variableOrder = \$Config.'poshCommit.variables'.psobject.properties.Name
    foreach (\$varName in \$variableOrder) {
        \$varItems = \$Config.'poshCommit.variables'.\$varName
        \$title = "Select a \$(\$varName):"

        \$selectedLabel = Out-PoshSelectionMenu -Items \$varItems -Title \$title
        if (\$null -eq \$selectedLabel) {
            Write-Host "Commit cancelled." -ForegroundColor Red
            return \$null # User cancelled
        }
        \$selections[\$varName] = \$selectedLabel
    }

    # --- Message Input ---
    \$message = Read-Host "Enter commit message (subject)"
    if ([string]::IsNullOrWhiteSpace(\$message)) {
        Write-Host "Commit cancelled due to empty message." -ForegroundColor Red
        return \$null
    }
    \$selections['message'] = \$message.Trim()


    # --- Message Construction ---
    \$commitMessage = \$Config.'poshCommit.template'[0]

    # Handle special optional scope case first
    if (\$selections.ContainsKey('scope') -and [string]::IsNullOrEmpty(\$selections['scope'])) {
        \$commitMessage = \$commitMessage -replace "\(\{scope\}\)", ""
    }

    # Perform all other replacements
    foreach (\$key in \$selections.Keys) {
        \$placeholder = "{" + \$key + "}"
        \$commitMessage = \$commitMessage.Replace(\$placeholder, \$selections[\$key])
    }

    # Clean up any extra whitespace
    \$commitMessage = \$commitMessage -replace '\s+', ' '

    return \$commitMessage.Trim()
}
"@


# --- 3. Create Files and Write Content ---
Write-Host "Creating module files..."

try {
    @{
        (Join-Path $baseDir "PoshCommit.psd1") = $psd1Content;
        (Join-Path $baseDir "PoshCommit.psm1") = $psm1Content;
        (Join-Path $configDir "poshCommit.json") = $jsonContent;
        (Join-Path $publicDir "Invoke-PoshCommit.ps1") = $invokePoshCommitContent;
        (Join-Path $publicDir "Get-PoshCommitMessage.ps1") = $getPoshCommitMessageContent;
        (Join-Path $privateDir "Get-PoshCommitConfig.ps1") = $getPoshCommitConfigContent;
        (Join-Path $privateDir "Out-PoshSelectionMenu.ps1") = $outPoshSelectionMenuContent;
        (Join-Path $privateDir "Invoke-GitCommand.ps1") = $invokeGitCommandContent;
        (Join-Path $privateDir "New-PoshCommitMessage.ps1") = $newPoshCommitMessageContent;
    }.GetEnumerator() | ForEach-Object {
        Write-Host "  - Creating file: $($_.Name)"
        Set-Content -Path $_.Name -Value $_.Value -Encoding UTF8 -Force
    }
    Write-Host "All files created successfully." -ForegroundColor Green
}
catch {
    Write-Error "An error occurred while creating a file: $_"
    return
}

# --- 4. Final Instructions ---
Write-Host "`n--- PoshCommit Module Created Successfully ---" -ForegroundColor Cyan
Write-Host "To install the module, choose ONE of the following options:"
Write-Host "1) For the current user:"
Write-Host "   Copy the entire '.\PoshCommit' folder to one of your module paths."
Write-Host "   You can find your paths by running: \$env:PSModulePath -split ';'"
Write-Host "   A common path is: '$([Environment]::GetFolderPath('MyDocuments'))\PowerShell\Modules'"
Write-Host ""
Write-Host "2) For temporary use in the current session:"
Write-Host "   Run the following command:"
Write-Host "   Import-Module -Name '$(Resolve-Path $baseDir)'"
Write-Host ""
Write-Host "Once imported, you can run 'Invoke-PoshCommit' inside a Git repository."
```