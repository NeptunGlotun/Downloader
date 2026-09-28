# Threat Model — Downloader Application

| Attribute | Value |
| --- | --- |
| Document Title | Threat Model & STRIDE Analysis — Downloader |
| Version | 1.0.0 |
| Status | Active |
| Last Updated | 2026-09-28 |
| Owner | NeptunGlotun |
| Related Documents | `security_policy.md`, `business_logic.md` |

## 1. Scope
This Threat Model covers:
- Desktop WPF UI client interaction.
- Download and Transcode execution processes (`yt-dlp` & `FFmpeg` wrappers).
- Local file system operations (saving media, reading configuration).
- Network communications with remote media streaming servers.

**Out of Scope:** Physical tampering with local user hardware, OS kernel compromise, and internal server-side infrastructure of target video hosting services.

## 2. Attacker Model (Threat Actors)

| ID | Attacker Type | Access Level | Capabilities & Motivations |
| --- | --- | --- | --- |
| **TA-01** | Untrusted Web Resource / Malicious Server | External / Network | Controls response payloads, HTTP headers, or crafted media streams to trigger parser exploits/buffer overflows. |
| **TA-02** | Local Malicious User / Local Malware | Local User Privilege | Injects malicious parameters into media URLs/file paths to execute arbitrary commands or overwrite files. |
| **TA-03** | Man-in-the-Middle (MitM) Attacker | Network Path | Attempts to intercept, modify, or block media downloads or CLI engine updates over unencrypted connections. |

## 3. Data Flow Diagram (DFD) & System Elements

### DFD Elements

| ID | DFD Type | Element Name | Description |
| --- | --- | --- | --- |
| **E1** | External Interactor | User | Local desktop user interacting with WPF UI. |
| **E2** | External Interactor | Remote Media Hosting | Video/audio streaming platforms (e.g., YouTube, Vimeo). |
| **P1** | Process | WPF UI Client App | UI layer handling user input, options, and status display. |
| **P2** | Process | Execution Engine | Application core managing `yt-dlp` and `FFmpeg` CLI invocations. |
| **D1** | Data Store | Local File System | Output media directory and local settings storage. |
| **F1** | Data Flow | User -> WPF UI | Input URL, format presets, path selection. |
| **F2** | Data Flow | WPF UI -> Execution Engine | Internal domain commands (`StartConversionAC`, `StartDownloadAC`). |
| **F3** | Data Flow | Execution Engine -> Remote Media Hosting | HTTPS media stream requests and metadata queries. |
| **F4** | Data Flow | Execution Engine -> Local File System | Writing downloaded chunks and transcoded media files. |
| **TB1** | Trust Boundary | User / UI Boundary | Boundary separating untrusted user input from application domain. |
| **TB2** | Trust Boundary | Local System / Network Boundary | Boundary separating internal application logic from external network servers. |

## 4. STRIDE Threat Matrix

Canonical STRIDE mapping per element type:
- **Process (P):** S, T, R, I, D, E
- **Data Flow (F):** T, I, D
- **Data Store (D):** T, I, D
- **External Interactor (E):** S, R

| Element | Threat Category | Threat Scenario | Impact | Mitigation Strategy |
| --- | --- | --- | --- | --- |
| **E1 User** *(External)* | **R** (Repudiation) | User denies initiating a specific network download that violates local network policies. | Low | Maintain local local-only timestamped execution log. |
| **F1 User -> UI** *(Data Flow)* | **T** (Tampering) | Attacker injects shell command delimiters into URL input field. | High | Strict URL format validation using `Uri.TryCreate` and domain whitelisting. |
| **P1 WPF UI** *(Process)* | **E** (Elevation of Privilege) | Unhandled UI exceptions lead to application crash or unauthorized privilege escalation via process inheritance. | Medium | Enforce global exception handling; run process strictly with standard user privileges. |
| **P2 Exec Engine** *(Process)* | **T** (Tampering) | CLI command strings constructed via simple string concatenation allow Command Injection into `yt-dlp`/`FFmpeg`. | **Critical** | Use `ProcessStartInfo.ArgumentList` instead of raw string arguments; bypass command shell execution. |
| **P2 Exec Engine** *(Process)* | **D** (Denial of Service) | Downloading ultra-large media files or infinite streams exhausts local disk space and CPU resources. | High | Implement maximum file size limits, timeout thresholds, and background cancellation tokens. |
| **F3 Engine -> Media Hosting** *(Data Flow)* | **I** (Info Disclosure) | Media stream requests sent via unencrypted HTTP expose user viewing activity to network eavesdroppers. | Medium | Enforce mandatory HTTPS for all remote network calls (`TB2`). |
| **D1 Local Storage** *(Data Store)* | **T** (Tampering) | Path Traversal attack (`../../Windows/System32/`) overwrites critical system files during file saving. | **Critical** | Sanitize output filenames; strictly resolve paths using `Path.GetFullPath` and enforce write-destination directory boundaries. |
| **D1 Local Storage** *(Data Store)* | **I** (Info Disclosure) | Error logs store sensitive URLs containing embedded auth tokens or credentials. | Medium | Strip URL query parameters containing tokens before writing to local log files. |
