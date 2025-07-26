## **Specification for PowerShell Git Commit Message Module (`PoshCommit`)**

**1. Module Name & Core Cmdlets**
*   **Module Name:** `PoshCommit`
*   **Primary Cmdlets:**
    *   `Invoke-PoshCommit`: Orchestrates interactive selection, message generation, and Git commit.
    *   `Get-PoshCommitMessage`: Generates and outputs the commit message string without Git operations.

**2. Configuration File Specification**

*   **File Name:** `poshCommit.json` (or potentially `settings.json` if preferred for VS Code extension compatibility).
*   **Loading Order:**
    1.  `.\poshCommit.json` (or `.\.poshCommit.json` or `.\settings.json`) in the current Git repository root.
    2.  `$PoshHome\Modules\PoshCommit\poshCommit.json` (where `$PoshHome` is typically `~\Documents\PowerShell` for the current user).
    3.  Internal default configuration (if no file found).
*   **Format:** JSON
*   **Schema (based on user's input `settings.json`):**

    ```json
    {
      "poshCommit.variables": {
        "emoji": [
          { "label": ":construction:", "detail": "WIP" },
          { "label": ":books:", "detail": "Documentation" },
          // ... more emoji objects
        ],
        "feat": [
          { "label": "feat", "detail": "Feature" },
          { "label": "bugfix", "detail": "Bug Fix" },
          // ... more feat objects
        ],
        "scope": [
          { "label": "sites", "detail": "Site Map" },
          { "label": "deps", "detail": "Dependencies" },
          { "label": "", "detail": "No Scope" } // Added for explicit "No Scope" option
          // ... more scope objects
        ]
      },
      "poshCommit.template": [
        "{emoji} {feat}({scope}): {message}"
      ]
    }
    ```
    *   `poshCommit.variables`: Contains arrays named after the placeholders (e.g., `emoji`, `feat`, `scope`).
        *   Each object in these arrays MUST have a `label` property (the string to be inserted) and a `detail` property (a description for the user).
    *   `poshCommit.template`: An array containing one string, which is the commit message template. Placeholders are enclosed in curly braces (`{placeholderName}`).

**3. Core Logic and Workflow (`Invoke-PoshCommit` Cmdlet)**

*   **3.1 Configuration Loading:**
    *   Use `ConvertFrom-Json` to parse the configuration file.
    *   Store `poshCommit.variables` and `poshCommit.template` in module-level variables.
*   **3.2 Git Repository Check:**
    *   Execute `git rev-parse --is-inside-work-tree` to check if inside a Git repository.
    *   If not, display a warning and exit for `Invoke-PoshCommit`. `Get-PoshCommitMessage` can proceed.
*   **3.3 Interactive Selection (`Out-PoshSelectionMenu` function - internal):**
    *   A reusable function will be created to present interactive menus.
    *   **Input:** List of objects (e.g., from `poshCommit.variables.emoji`), title, prompt message.
    *   **Display:** Loop through items, showing `label` and `detail`. Use `[Console]::ReadKey()` to capture key presses.
    *   **Navigation:** Arrow keys for selection, Enter to confirm.
    *   **Output:** The `label` property of the selected item.
*   **3.4 User Input:**
    *   Use `Read-Host` to prompt for the main commit message.
*   **3.5 Message Construction Logic:**
    *   Retrieve the template string: `$template = $config.'poshCommit.template'[0]`.
    *   Perform direct replacements for mandatory parts:
        *   `$commitMessage = $template.Replace('{emoji}', $selectedEmojiLabel)`
        *   `$commitMessage = $commitMessage.Replace('{feat}', $selectedFeatLabel)`
        *   `$commitMessage = $commitMessage.Replace('{message}', $userMessage)`
    *   **Special Handling for Optional Components (e.g., Scope):**
        *   If the selected `$selectedScopeLabel` is empty (`""`):
            *   Use a regular expression to find and remove `({scope})` from `$commitMessage`.
            *   Example: `$commitMessage = $commitMessage -replace "`\({scope}\)`", ""`
            *   Then, replace any remaining `{scope}` with an empty string: `$commitMessage = $commitMessage.Replace('{scope}', '')`.
        *   If the selected `$selectedScopeLabel` is NOT empty:
            *   `$commitMessage = $commitMessage.Replace('{scope}', $selectedScopeLabel)`
*   **3.6 Git Operations (for `Invoke-PoshCommit` only):**
    *   Execute `git.exe add -A` (to stage all changes).
    *   Execute `git.exe commit -m "$commitMessage"` (using the generated message).
    *   Capture and display Git command output/errors.

**4. Error Handling**

*   **Configuration:**
    *   Catch `ConvertFrom-Json` errors for invalid JSON.
    *   Validate the presence of required keys (`poshCommit.variables`, `poshCommit.template`). If missing, log warnings and potentially use fallback defaults.
*   **Git:**
    *   Handle non-zero exit codes from `git.exe` commands.
    *   Provide user-friendly messages if `git.exe` is not found in the PATH.
*   **User Input:**
    *   Ensure robust handling of user cancelling selections or providing empty message (e.g., allow or re-prompt).

**5. Implementation Details**

*   **PowerShell Version:** Develop targeting PowerShell 7+, ensuring backward compatibility with 5.1 where possible.
*   **Module Structure:**
    *   `PoshCommit.psd1` (Module Manifest)
    *   `PoshCommit.psm1` (Main Module File, containing functions and cmdlets)
    *   `Public\` (for exported functions/cmdlets)
    *   `Private\` (for internal helper functions)
    *   `Config\` (for default `poshCommit.json`)
*   **Interactive Menus:** Implement a custom console-based menu function in a private helper function.
*   **Git Execution:** Use `Start-Process -FilePath "git.exe" -ArgumentList ... -NoNewWindow -Wait -PassThru` to execute Git commands and capture output.
*   **Default Configuration:** Embed a default `poshCommit.json` structure directly in the module or include it as a separate file within the module directory.

**6. Example Usage**

```powershell
# Interactive selection, stages changes, and commits
Invoke-PoshCommit

# Generates and prints the commit message string only
$myCommitMsg = Get-PoshCommitMessage
Write-Host "Generated Commit Message: $myCommitMsg"
```