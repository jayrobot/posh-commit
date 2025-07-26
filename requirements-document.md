Here are the requirements and specifications for your PowerShell module, designed to mimic the VS Code Git Commit Message extension based on your provided `settings.json` file.

---

## **Requirements Document for PowerShell Git Commit Message Module (`PoshCommit`)**

**1. Introduction**
This document outlines the functional and non-functional requirements for a PowerShell module, tentatively named `PoshCommit`, which aims to streamline the process of creating Git commit messages. The module will mimic the "VSCode Git Commit Message" extension by allowing users to interactively select predefined components (emoji, feature/type, scope) and compose a commit message based on a user-configurable template.

**2. Functional Requirements (FR)**

*   **FR.1 Configuration Management:**
    *   **FR.1.1 Config File Reading:** The module MUST read its configuration from a JSON file.
        *   **FR.1.1.1:** It should first look for a configuration file (e.g., `poshCommit.json` or `settings.json`) in the root of the current Git repository.
        *   **FR.1.1.2:** If no project-level configuration is found, it SHOULD look for a default configuration file in a user-specific PowerShell module directory (e.g., `~\Documents\PowerShell\Modules\PoshCommit\poshCommit.json`).
        *   **FR.1.1.3:** If no configuration file is found at all, the module SHOULD fall back to a reasonable default internal configuration, providing a warning to the user.
    *   **FR.1.2 Config Structure:** The configuration file MUST adhere to a defined JSON structure, specifying:
        *   **FR.1.2.1 Variables:** Lists of available emojis, commit types (`feat`), and scopes, each with a `label` (the value to be inserted) and a `detail` (a human-readable description).
        *   **FR.1.2.2 Template:** A string (or an array of strings to be concatenated) defining the structure of the final commit message, using placeholders (e.g., `{emoji}`, `{feat}`, `{scope}`, `{message}`).

*   **FR.2 Interactive User Interface:**
    *   **FR.2.1 Emoji Selection:** The module MUST present an interactive menu allowing the user to select an emoji from the configured list. The menu should display both the emoji's `label` and `detail`.
    *   **FR.2.2 Feature (Type) Selection:** The module MUST present an interactive menu allowing the user to select a commit type (feature) from the configured list. The menu should display both the `label` and `detail`.
    *   **FR.2.3 Scope Selection:** The module MUST present an interactive menu allowing the user to select a scope from the configured list. The menu should display both the `label` and `detail`. This menu SHOULD include an option to select no scope (e.g., an empty `label`).
    *   **FR.2.4 Message Input:** The module MUST prompt the user to enter the main commit message/subject line.
    *   **FR.2.5 Navigation:** All interactive menus SHOULD support standard console navigation (e.g., up/down arrow keys to move, Enter to select).

*   **FR.3 Commit Message Generation:**
    *   **FR.3.1 Template Substitution:** The module MUST construct the final commit message by substituting the selected emoji, feature, scope, and the entered message into the configured template.
    *   **FR.3.2 Optional Component Handling:** The module MUST intelligently handle optional components in the template. Specifically, if a placeholder like `{scope}` is surrounded by parentheses (e.g., `({scope})`) and the user selects an empty scope, the entire parenthesized section `()` MUST be omitted from the final message.

*   **FR.4 Git Integration:**
    *   **FR.4.1 Git Repository Detection:** The module MUST detect if it is being run within a Git repository. If not, Git-related operations should be skipped, and the user informed.
    *   **FR.4.2 Commit Action Cmdlet:** The module MUST provide a primary cmdlet (e.g., `Invoke-PoshCommit`) that stages all pending changes in the current Git repository (`git add .` or `git add -A`) and then commits them using the generated message.
    *   **FR.4.3 Message Output Cmdlet:** The module MUST provide a separate cmdlet (e.g., `Get-PoshCommitMessage`) that only generates and outputs the formatted commit message string, without performing any Git commit actions. This allows for flexibility and piping the output to other commands.

**3. Non-Functional Requirements (NFR)**

*   **NFR.1 Performance:** The module should provide a responsive user experience, with minimal noticeable delay during interactive selections and message generation.
*   **NFR.2 Usability:** The interactive menus and command-line interface should be intuitive and easy to use for PowerShell users.
*   **NFR.3 Compatibility:** The module MUST be compatible with PowerShell 5.1 and PowerShell Core (6.x, 7.x, and later versions).
*   **NFR.4 Robustness:** The module SHOULD gracefully handle malformed or incomplete configuration files by providing informative error messages and/or falling back to sensible defaults.
*   **NFR.5 Extensibility:** The configuration file format and module structure SHOULD be designed to allow for easy addition of new variable types or template components in the future.
*   **NFR.6 Documentation:** Comprehensive documentation MUST be provided, covering installation, configuration options, and usage examples for all cmdlets.

---