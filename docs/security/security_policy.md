# Security Policy — MediaConvert & Downloader

| Attribute | Value |
| --- | --- |
| Document Title | Security Policy — Downloader Project |
| Version | 1.0.0 |
| Status | Active |
| Classification | Internal |
| Last Updated | 2026-09-28 |
| Owner | NeptunGlotun |
| Related Documents | `project_policy.md`, `privacy_policy.md`, `threat_model.md` |

---

## 1. Purpose
This Security Policy defines mandatory requirements and guidelines for secure software development, execution environment, and lifecycle management for the **Downloader** application (C# / .NET 8 / WPF). Its goal is to minimize security vulnerabilities, prevent command injection during external tool execution (yt-dlp / FFmpeg), protect local data, and define vulnerability handling procedures.

## 2. Scope
This policy applies to:
- Source code in `src/` and dependencies within `.csproj`.
- Infrastructure configuration, build scripts, GitHub Actions/CI configuration, and `docs/`.
- Integration wrappers interacting with local CLI processes (`yt-dlp.exe`, `ffmpeg.exe`) and network streams.
- All contributors, manual code reviewers, and automated tools involved in project development.

## 3. Roles and Responsibilities
- **Project Owner / Lead Developer:** Approves security policies, handles risk acceptance, and authorizes exceptions.
- **Developer:** Ensures code compliance with input validation and safe execution guidelines.
- **Reviewer:** Conducts pull request checks, verifying that CLI commands and network calls are free from injection flaws.
- **Automated CI / Security Tooling:** Performs dependency scans and static code analysis prior to code merges.

## 4. Security Principles
1. **Least Privilege:** Internal process execution and file system access are restricted strictly to necessary user directories.
2. **Secure by Default:** Network downloads default to HTTPS. Executable path resolving strictly forbids searching untrusted working directories.
3. **Defense in Depth:** Input validation occurs at the UI boundary, application command layer, and process invocation layer.
4. **Fail Securely:** Exceptions during downloading or converting media terminate the operation safely without leaving orphaned system processes or exposed raw paths in error alerts.
5. **No Secrets in Repository:** API tokens, private SSH keys, and system credentials must never be committed to Git.

## 5. Secure Development Environment
1. **Repository Access & Git Workflow:** Direct commits to `main` are restricted. Changes must pass Pull Request code reviews.
2. **SSH Authentication:** Commit signed pushes and repository operations require SSH keys (`ed25519`).
3. **Dependency Integrity:** External NuGet packages and CLI binaries (`yt-dlp`, `FFmpeg`) must be verified for license compatibility and scanned for known vulnerabilities (CVEs).

## 6. Software Protection & Secure Implementation
1. **Command Injection Prevention:**
   - Raw user input (URLs, target filenames, resolution flags) must NEVER be concatenated directly into shell arguments.
   - External processes must be invoked via `System.Diagnostics.ProcessStartInfo` using explicit argument lists (`ArgumentList` property in .NET) without invoking `cmd.exe` or PowerShell context.
2. **File System Safety:**
   - Output paths must be validated to prevent Directory Traversal attacks (`../`).
   - File extensions are strictly matched against a domain whitelist (`.mp4`, `.mkv`, `.mp3`, `.aac`, `.webm`).
3. **Memory & Error Management:**
   - Internal stack traces and absolute local system paths are suppressed in user-facing error dialogs.

## 7. Security Verification
1. **Static Code Analysis:** Roslyn Analyzers and Security Code Scan are executed during build.
2. **Code Review Criteria:** Every PR modifying process execution, network operations, or file IO requires manual approval.
3. **Release Gate:** Releases with unaddressed high or critical CVE vulnerabilities in dependencies are strictly blocked.

## 8. Vulnerability Management
1. **Reporting:** Security defects are logged internally via private issue tracking.
2. **Remediation SLA:** Critical vulnerabilities affecting input parsing or command execution must be fixed within 7 days.
3. **Post-Mortem:** Every resolved security defect requires updating `threat_model.md` and adding regression tests.

## 9. Exceptions
Exceptions to this security policy require written justification, specifying compensating security controls, expiration date, and approval by the Project Owner.
