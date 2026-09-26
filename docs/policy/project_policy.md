# PROJECT SECURITY & DEVELOPMENT POLICY

## 1. General Provisions
This policy governs software development rules, architectural guidelines, and security requirements for the **MediaConvert & Downloader** project (C# / .NET 8 / WPF).
Compliance with this policy is mandatory for all contributors, adhering to ISO/IEC/IEEE 29148 and ISO/IEC/IEEE 42010 standards.

## 2. Security Policy
1. **No Secrets in Repository:** Private keys, API tokens, passwords, database credentials, and log files must never be committed to Git history.
2. **Input Validation & Sanitization:**
   - All user inputs (URLs, file paths) must undergo strict validation before passing to internal processes or CLI engines (such as FFmpeg or yt-dlp) to prevent **Command Injection** vulnerabilities.
3. **`.gitignore` Rules:** System build artifacts (`/bin/`, `/obj/`), user configurations, SSH keys (`.ssh/`), and temporary media downloads must be excluded from version control.

## 3. Architecture & Code Standards
1. **Clean Architecture / DDD Principles:**
   - Codebase is organized into **Domain**, **Application**, **Infrastructure**, and **UI (WPF/MVVM)** layers.
   - Domain entities and Value Objects must be immutable.
2. **Exception & Error Handling:**
   - Empty `catch` blocks are strictly forbidden.
   - All critical processing errors must be logged internally without exposing raw system file paths to the end user.

## 4. Git Workflow
1. **Commit Message Prefixes:**
   - `[INIT]` — Initial repository setup and base project creation.
   - `[DOCS]` — Documentation, specifications, and policy updates.
   - `[FEAT]` — Implementation of new features.
   - `[FIX]` — Bug fixes and security patches.
2. **Authentication:** Repository interactions are allowed exclusively via SSH protocol using `ed25519` keys.
