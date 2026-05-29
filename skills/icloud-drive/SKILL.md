---
name: icloud-drive
description: |
  Interact with iCloud Drive via its internal browser API  list files, browse
  folders, upload, download, create folders, move, rename, and trash files.
  Use when the user wants to automate iCloud Drive, manage files in iCloud,
  browse iCloud Drive folders, upload to iCloud, download from iCloud, organize
  iCloud files, or perform any iCloud Drive file management task. Activate on
  mentions of iCloud Drive, iCloud files, iCloud folders, Apple cloud storage,
  Documents folder, Desktop sync, or iCloud file operations.
allowed-tools: bash
---

# iCloud Drive

Control iCloud Drive in the browser via its internal docws/drivews APIs. Requires the user to be logged into icloud.com in a browser tab.

## Prerequisites

The user must be logged into iCloud Drive at `https://www.icloud.com/iclouddrive/` in the browser. The skill uses page-context evaluation to call the internal APIs with cookie-based auth.

If the iCloud tab is not open, the script will open one automatically and wait for login.

## Commands

```bash
icloud-drive list [<path>] [--max=<n>]           # List folder contents (default: root)
icloud-drive info <path-or-docwsid>              # Get file/folder metadata
icloud-drive upload <local-path> [<dest-path>]   # Upload a file
icloud-drive download <path-or-docwsid> [--out=<path>]  # Download file
icloud-drive mkdir <path>                        # Create a folder
icloud-drive move <path> --to=<dest-path>        # Move a file/folder
icloud-drive rename <path> --name=<new-name>     # Rename a file/folder
icloud-drive trash <path-or-docwsid>             # Move to trash
icloud-drive recent [--max=<n>]                  # List recently modified files
```

Paths use forward slash notation relative to iCloud Drive root, e.g.:
- `Documents/my-file.txt`
- `Desktop/project/README.md`
- `Downloads/archive.zip`

## Architecture

- **Auth**: Cookie-based (httpOnly cookies set by iCloud login flow)
- **List/Rename/Move/Trash**: `https://p{N}-drivews.icloud.com/` endpoints (POST with Content-Type: text/plain to avoid CORS preflight)
- **Upload/Download/Create**: `https://p{N}-docws.icloud.com/ws/com.apple.CloudDocs/` endpoints
- **Content delivery**: Pre-signed URLs on `*.icloud-content.com` (no auth needed)
- **Server prefix**: Extracted from page's network requests (e.g., `p54`)

All operations run from the iCloud.com page context to leverage first-party cookies.

## Notes

- iCloud Drive uses two API services: `drivews` (folder operations) and `docws` (file operations)
- The `drivewsid` format is `{TYPE}::com.apple.CloudDocs::{UUID}` (e.g., `FILE::com.apple.CloudDocs::ABC123`)
- Uploads are two-step: request upload URL ’ upload content ’ commit via update/documents
- Downloads generate time-limited pre-signed URLs
- The server prefix (p54, etc.) is session-specific
- Etags are required for rename/move/trash operations (fetched automatically)
