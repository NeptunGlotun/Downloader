# Privacy Policy — Downloader Application

| Attribute | Value |
| --- | --- |
| Document Title | Privacy Policy — Downloader Application |
| Version | 1.0.0 |
| Effective Date | 2026-09-28 |
| Status | Active / Public |

## 1. Overview
The **Downloader** application operates as a client-side media download and conversion utility. This Privacy Policy details what data is processed, stored, or transmitted when utilizing the desktop application.

## 2. Data Minimization Principle
The application is designed in compliance with privacy-by-default and data minimization principles. It does not collect, track, or commercialize personal user identification data.

## 3. Categories of Data Processed

| Category | Description | Storage Location | Retention / Scope |
| --- | --- | --- | --- |
| **Media URLs** | User-provided URLs for video/audio extraction. | Transient memory / Local download log. | Processed strictly during download operation; stored locally if history is enabled. |
| **Download Destination Paths** | Selected local directory paths for saving media files. | Local app configuration (`appsettings.json` / Registry). | Stored locally on user device. Never transmitted externally. |
| **Media Conversion Presets** | User-configured bitrate, format, and resolution preferences. | Local application settings. | Persisted locally across app sessions. |
| **Application Logs** | Local error logs for debugging failed conversions. | Local user `%APPDATA%\Downloader\logs`. | Stored locally; strictly excludes user telemetry or personal credentials. |

## 4. Third-Party Interactions & Network Requests
1. **External Media Servers:** When initiating a download, direct HTTPS requests are dispatched to target media hosting platforms (e.g., YouTube, Vimeo) to fetch stream metadata and media fragments.
2. **CLI Engine Updates:** If automatic updates for `yt-dlp` are enabled, the application contacts official GitHub releases repositories to fetch verified executable binaries.
3. **No Telemetry / Analytics:** The application does not incorporate third-party analytics SDKs, advertising IDs, or user tracking telemetry.

## 5. Storage and Security
- All user preferences and conversion histories remain exclusively on the user's local file system.
- Users can clear local history and log files at any time by deleting the application cache directory.

## 6. User Rights and Control
Users retain full control over their data:
- **Right to Access & Erase:** Users can inspect or delete local configuration files and download logs at any time without network reliance.
- **Network Control:** The application operates completely offline when performing offline media transcoding via local FFmpeg.
