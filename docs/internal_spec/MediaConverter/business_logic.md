# ====== MediaConverter BUSINESS LOGIC ======

## Purpose

`MediaConverter` owns:
- Media file transcoding concepts and operations (codecs, bitrates, containers, resolution).
- Transcoding parameters validation and CLI command construction for the processing engine (FFmpeg).
- Execution tracking and status management for conversion jobs.

**NOT here:**
- Network downloading logic — `[MediaDownloader]`.
- Graphical UI rendering — `[WPF.UI]`.

## Entities

| Entity | Basic Fields | Description | Invariants |
|---|---|---|---|
| `ConversionJob` *(Aggregate Root)* | uuid, sourcePath, targetPath, status, presetRef | Represents a single media transcoding job. | - Identity (`uuid`) is immutable.<br>- Status changes strictly via lifecycle policy.<br>- `sourcePath` must exist on the local file system before start. |
| `MediaPreset` *(Entity)* | uuid, containerFormat, audioCodec, videoCodec, bitrate | Preset configuration for output media format. | - `containerFormat` cannot be empty.<br>- `bitrate` must be strictly > 0. |

## Status Lifecycle

- Newly created job always starts in `QUEUED` status.

Allowed transitions:

| From | Self-initiated | System/admin-initiated |
|---|---|---|
| `QUEUED` | → `PROCESSING` | → `CANCELLED` |
| `PROCESSING` | → `CANCELLED` | → `COMPLETED`, `FAILED` |
| `COMPLETED` | — (terminal) | — (terminal) |
| `FAILED` | — (terminal) | — (terminal) |

## Value Objects

| Value Object | Description | Invariants |
|---|---|---|
| `JobUuidVO` | Strongly-typed unique job identifier. | - Immutable.<br>- Valid UUID v4 format. |
| `MediaFormatVO` | Wraps output extension (e.g., `.mp3`, `.mp4`, `.mkv`). | - Immutable.<br>- Whitelisted formats only. |
| `BitrateVO` | Encapsulates audio/video bitrate in kbps. | - Value within range [64..50000]. |

## Domain Policies

| Domain Policy | Description |
|---|---|
| `ValidFormatDPolicy` | Validates compatibility between selected audio/video codecs and output container format. |
| `ResourceLimitDPolicy` | Limits parallel conversion processes to prevent CPU exhaustion. |

## Domain Services

| Domain Service | Operation |
|---|---|
| `TranscodeMediaService` | Validates jobs via `ValidFormatDPolicy`, prepares execution arguments, and triggers domain events. |

## Domain Events

| Event | Carries | Notes |
|---|---|---|
| `JobStartedDE` | job uuid, sourcePath, targetPath | Emitted upon transition to `PROCESSING`. |
| `JobCompletedDE` | job uuid, executionTime | Emitted upon successful conversion. |
| `JobFailedDE` | job uuid, errorMessage | Emitted on FFmpeg process failure. |

## Application Commands & Queries

**Commands (`AC`):**
- `StartConversionAC` — Schedules and starts a media conversion process.
- `CancelJobAC` — Aborts an ongoing conversion task.

**Queries (`AQ`):**
- `GetJobStatusAQ` — Returns current job status and progress percentage.

## Infrastructure

### Models
- `ConversionJobModel` — Persistence model for logging executed jobs into local storage or cache.
