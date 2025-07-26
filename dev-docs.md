## **Developer Documentation**

### **1. Project Structure**

The `PoshCommit` module adheres to a standard PowerShell module structure:

```
PoshCommit/
├── PoshCommit.psd1           # Module Manifest
├── PoshCommit.psm1           # Main Module Script (contains functions, cmdlets)
├── Public/                   # Contains exported functions/cmdlets
│   ├── Invoke-PoshCommit.ps1
│   └── Get-PoshCommitMessage.ps1
├── Private/                  # Contains internal helper functions
│   ├── Get-PoshCommitConfig.ps1
│   ├── Out-PoshSelectionMenu.ps1
│   └── Invoke-GitCommand.ps1
├── Config/                   # Default configuration files
│   └── poshCommit.json       # Default configuration
└── README.md
```

*   **`PoshCommit.psd1`**: Defines the module, its exported functions, version, author, etc. This is crucial for PowerShell to recognize and load the module.
*   **`PoshCommit.psm1`**: The primary script that loads all other `.ps1` files (from `Public` and `Private` directories) and exports the public functions.
*   **`Public/`**: Holds the PowerShell script files for each public-facing cmdlet (functions exposed to the user).
*   **`Private/`**: Holds helper functions that are not directly exposed to the user but are used internally by the public cmdlets.
*   **`Config/poshCommit.json`**: The default configuration file that ships with the module.

### **2. Core Logic Explained**

#### **2.1 Configuration Loading (`Private\Get-PoshCommitConfig.ps1`)**

This function is responsible for finding and parsing the `poshCommit.json` configuration.

*   **Logic:**
    1.  Determines the current Git repository root (if any) using `git rev-parse --show-toplevel`.
    2.  Checks for `poshCommit.json`, `.poshCommit.json`, or `settings.json` within the repository root.
    3.  If not found, it checks the module's `Config` directory for `poshCommit.json`.
    4.  If still not found, it falls back to a hardcoded default JSON string, logging a warning.
    5.  Uses `ConvertFrom-Json` to parse the file content.
    6.  Performs basic validation to ensure `poshCommit.variables` and `poshCommit.template` exist.

#### **2.2 Interactive Menu (`Private\Out-PoshSelectionMenu.ps1`)**

This is a generic helper function to present console-based selection menus.

*   **Input:**
    *   `Items`: An array of objects, each with `label` and `detail` properties.
    *   `Title`: A string for the menu title.
    *   `Prompt`: A string for the prompt message.
*   **Logic:**
    *   Uses `[Console]::CursorTop` and `[Console]::CursorLeft` to control cursor position.
    *   Renders the list of items, highlighting the currently selected item.
    *   Captures key presses using `[Console]::ReadKey($true)` (to prevent outputting the key itself).
    *   Handles `UpArrow`, `DownArrow`, `Enter` (for selection), and `Escape` (for cancellation).
    *   Clears the rendered menu lines after selection to keep the console clean.
*   **Output:** The `label` property of the selected item.

#### **2.3 Commit Message Generation**

This logic is typically found within both `Invoke-PoshCommit.ps1` and `Get-PoshCommitMessage.ps1` (or a shared private helper function).

*   **Steps:**
    1.  Call `Out-PoshSelectionMenu` for emoji, `feat`, and `scope`.
    2.  Use `Read-Host` to get the user's message.
    3.  Retrieve the template string from the loaded configuration.
    4.  **Direct Substitution:** Replace `{emoji}`, `{feat}`, and `{message}` placeholders with their respective selected values.
    5.  **Optional Scope Handling:** This is critical for clean messages.
        *   If the selected scope `label` is empty:
            *   A regular expression is used to find and remove the entire `({scope})` pattern from the template.
            *   Example: `$commitMessage = $commitMessage -replace "`\({scope}\)`", ""`
        *   If the selected scope `label` is NOT empty:
            *   The `{scope}` placeholder is replaced with the selected scope label.

#### **2.4 Git Integration (`Private\Invoke-GitCommand.ps1`)**

A helper function to safely execute Git commands.

*   **Logic:**
    *   Uses `Start-Process -FilePath "git.exe" -ArgumentList $args -NoNewWindow -Wait -PassThru`
    *   Captures `StandardOutput` and `StandardError` streams.
    *   Checks the `ExitCode` for success/failure.
    *   Provides meaningful error messages if `git.exe` is not found or the command fails.

### **3. Adding New Variables and Templates**

Extending `PoshCommit` is straightforward due to its configuration-driven design.

1.  **Modify `poshCommit.json`:**
    *   **New Variable Type:** Add a new key-value pair under `poshCommit.variables`. The value should be an array of `{"label": "", "detail": ""}` objects.
        ```json
        "newType": [
          { "label": "label1", "detail": "Description 1" },
          { "label": "label2", "detail": "Description 2" }
        ]
        ```
    *   **Update Template:** Add the new placeholder to `poshCommit.template`.
        ```json
        "poshCommit.template": [
          "{emoji} {feat}({scope})[{newType}]: {message}"
        ]
        ```

2.  **Modify Cmdlets (`Invoke-PoshCommit.ps1`, `Get-PoshCommitMessage.ps1`):**
    *   After loading the configuration, call `Out-PoshSelectionMenu` for your new variable type.
        ```powershell
        $selectedNewType = Out-PoshSelectionMenu -Items $Config.'poshCommit.variables'.newType -Title "Select a new type" -Prompt "New Type:"
        ```
    *   Add a replacement step for your new placeholder *before* the `{message}` replacement. Consider if it needs special optional handling like `scope`.
        ```powershell
        # Assuming no optional handling like scope for newType
        $commitMessage = $commitMessage.Replace('{newType}', $selectedNewType)
        ```
    *   **Important:** Remember the order of replacements. Placeholders for optional components (like `({scope})`) should be processed *before* simple replacements to ensure the entire optional block is handled correctly if the inner value is empty.

### **4. Testing**

*   **Unit Testing:** PowerShell has Pester for unit testing.
    *   Write Pester tests for individual functions (e.g., `Get-PoshCommitConfig` for various file scenarios, `Out-PoshSelectionMenu` mock inputs, message generation logic for different templates and selections).
    *   Mock `git.exe` calls to avoid actual Git operations during tests.
*   **Integration Testing:**
    *   Test `Invoke-PoshCommit` and `Get-PoshCommitMessage` in a real Git repository (perhaps a temporary one created for tests).
    *   Verify the final commit message against expected output.
    *   Verify Git commands are executed correctly (e.g., check `git log` after `Invoke-PoshCommit`).

### **5. Contribution Guidelines**

If you'd like to contribute to `PoshCommit`, please follow these guidelines:

1.  **Fork the Repository:** Start by forking the main repository on GitHub.
2.  **Create a Branch:** Create a new branch for your feature or bug fix: `git checkout -b feature/your-feature-name` or `git checkout -b bugfix/issue-description`.
3.  **Code Style:** Adhere to common PowerShell best practices and the existing code style. Use `PSScriptAnalyzer` to check for style consistency.
4.  **Tests:** Add or update Pester tests for any new features or bug fixes. Ensure all existing tests pass.
5.  **Documentation:** Update this documentation if your changes affect user-facing features, configuration, or developer guidelines.
6.  **Commit Messages:** Use `PoshCommit` itself to create clean and consistent commit messages!
7.  **Pull Request:** Submit a pull request to the `main` branch of the upstream repository, describing your changes clearly.