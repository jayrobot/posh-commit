Here is the user and developer documentation for the `PoshCommit` PowerShell module.

---

# **PoshCommit PowerShell Module Documentation**

## **User Documentation**

### **1. Introduction**

`PoshCommit` is a PowerShell module designed to streamline your Git commit message creation process. Inspired by the VS Code Git Commit Message extension, it provides an interactive console experience to help you construct well-formatted commit messages based on a customizable template and predefined options for emojis, commit types (features), and scopes. This helps maintain consistent and clear commit history in your projects.

### **2. Features**

*   **Interactive Selection:** Easily choose emojis, commit types, and scopes from a list using console menus.
*   **Configurable Template:** Define your own commit message structure using placeholders.
*   **Customizable Variables:** Add or modify the lists of emojis, types, and scopes via a JSON configuration file.
*   **Smart Scope Handling:** Automatically omits parentheses around the scope if no scope is selected, keeping your messages clean.
*   **Git Integration:** Directly stage all changes and commit with the generated message within a Git repository.
*   **Flexible Output:** Generate just the commit message string for use in other scripts or commands.

### **3. Installation**

To install `PoshCommit`, you will typically place the module files in your PowerShell modules directory.

1.  **Download the Module:**
    *   Once the module is developed, you would typically download it from a repository (e.g., GitHub) or a PowerShell Gallery. For now, assume you have the `PoshCommit` folder containing `PoshCommit.psd1`, `PoshCommit.psm1`, etc.

2.  **Place in Module Path:**
    *   Find your PowerShell module path by running:
        ```powersell
        $env:PSModulePath -split ';'
        ```
    *   Choose a path (e.g., `C:\Users\<YourUser>\Documents\PowerShell\Modules` or `/usr/local/share/powershell/Modules` on Linux/macOS).
    *   Create a folder named `PoshCommit` inside one of these paths (e.g., `C:\Users\<YourUser>\Documents\PowerShell\Modules\PoshCommit`).
    *   Copy all the module files (including `PoshCommit.psd1`, `PoshCommit.psm1`, and any `Config` subfolder with `poshCommit.json`) into this `PoshCommit` folder.

3.  **Import the Module:**
    *   Open a new PowerShell session. The module should be discoverable. You can explicitly import it:
        ```powershell
        Import-Module PoshCommit
        ```
    *   To verify, run:
        ```powershell
        Get-Command -Module PoshCommit
        ```
        You should see `Invoke-PoshCommit` and `Get-PoshCommitMessage`.

### **4. Configuration**

`PoshCommit` uses a JSON configuration file, `poshCommit.json`, to define its behavior.

*   **Configuration File Location (Precedence Order):**
    1.  **Project-level:** `poshCommit.json` (or `.poshCommit.json` or `settings.json`) in the root of your current Git repository. This allows per-project customization.
    2.  **Global-level:** `poshCommit.json` located within the `PoshCommit` module directory (e.g., `C:\Users\<YourUser>\Documents\PowerShell\Modules\PoshCommit\Config\poshCommit.json`). This acts as a default for all projects.
    3.  **Internal Default:** If no file is found, the module falls back to a hardcoded internal default configuration.

*   **Configuration File Structure (`poshCommit.json` example):**

    ```json
    {
      "poshCommit.variables": {
        "emoji": [
          { "label": ":construction:", "detail": "WIP" },
          { "label": ":books:", "detail": "Documentation" },
          { "label": ":sparkles:", "detail": "New Feature" },
          { "label": ":bug:", "detail": "Bug Fix" }
        ],
        "feat": [
          { "label": "feat", "detail": "Feature" },
          { "label": "bugfix", "detail": "Bug Fix" },
          { "label": "refactor", "detail": "Refactor" },
          { "label": "chore", "detail": "Chore" }
        ],
        "scope": [
          { "label": "app", "detail": "App-Wide" },
          { "label": "ui", "detail": "User Interface" },
          { "label": "api", "detail": "API Endpoints" },
          { "label": "", "detail": "No Scope" }
        ]
      },
      "poshCommit.template": [
        "{emoji} {feat}({scope}): {message}"
      ]
    }
    ```

    *   **`poshCommit.variables`**: An object containing lists of options.
        *   Each key (e.g., `emoji`, `feat`, `scope`) corresponds to a placeholder in your template.
        *   Each value is an array of objects, where each object has:
            *   `"label"`: The string that will be inserted into the commit message.
            *   `"detail"`: A descriptive string shown in the interactive menu.
        *   **Important for Scope:** To allow for an optional scope, include an entry with an empty `label` and a descriptive `detail` like `{"label": "", "detail": "No Scope"}`.
    *   **`poshCommit.template`**: An array containing a single string that defines your commit message format.
        *   Placeholders are enclosed in curly braces, e.g., `{emoji}`, `{feat}`, `{scope}`, `{message}`.
        *   `{message}` is reserved for the user-entered subject line.
        *   If a placeholder like `{scope}` is wrapped in parentheses `({scope})` in the template and the user selects an empty scope, the entire `()` will be removed from the final message.

### **5. Usage**

`PoshCommit` provides two main cmdlets:

#### **`Invoke-PoshCommit`**

This is the primary cmdlet for interactive commit message creation and performing the Git commit.

*   **Behavior:**
    1.  Checks if you are in a Git repository. If not, it will warn you and exit.
    2.  Presents interactive menus for emoji, commit type, and scope selection.
    3.  Prompts for your main commit message.
    4.  Constructs the commit message based on your selections and the configured template.
    5.  Executes `git add -A` to stage all changes in your repository.
    6.  Executes `git commit -m "Your Generated Message"` to commit the staged changes.

*   **Syntax:**
    ```powershell
    Invoke-PoshCommit
    ```

*   **Example Walkthrough:**
    ```powershell
    PS C:\path\to\my\repo> Invoke-PoshCommit
    ```    *   **Select Emoji:**
        ```
        Select an emoji:
        > :sparkles: New Feature
          :bug: Bug Fix
          :books: Documentation
          :construction: WIP
        ```        (Use arrow keys, press Enter for `:sparkles:`)
    *   **Select Feature:**
        ```
        Select a commit type (feature):
        > feat Feature
          bugfix Bug Fix
          refactor Refactor
          chore Chore
        ```
        (Use arrow keys, press Enter for `feat`)
    *   **Select Scope:**
        ```
        Select a scope:
        > app App-Wide
          ui User Interface
          api API Endpoints
          No Scope
        ```
        (Use arrow keys, press Enter for `ui`)
    *   **Enter Message:**
        ```
        Enter commit message: Add interactive commit tool
        ```
        (Type your message and press Enter)

    *   **Result:**
        The module will then execute:
        `git add -A`
        `git commit -m ":sparkles: feat(ui): Add interactive commit tool"`
        You will see the standard Git commit output.

#### **`Get-PoshCommitMessage`**

This cmdlet generates and outputs the formatted commit message string *without* performing any Git operations. This is useful for piping the message to other commands or for previewing.

*   **Behavior:**
    1.  Presents interactive menus for emoji, commit type, and scope selection.
    2.  Prompts for your main commit message.
    3.  Constructs the commit message based on your selections and the configured template.
    4.  Outputs the final commit message string to the console or as an object.

*   **Syntax:**
    ```powershell
    Get-PoshCommitMessage
    ```

*   **Example Usage:**
    ```powershell
    PS C:\path\to\my\repo> $message = Get-PoshCommitMessage
    PS C:\path\to\my\repo> Write-Host "Your generated message is: '$message'"
    Your generated message is: ':bug: bugfix(api): Fix authentication issue'
    PS C:\path\to\my\repo> git commit -m "$message" # You can use it manually
    ```

---