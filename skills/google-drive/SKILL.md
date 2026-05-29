---
name: google-drive
description: |
  Interact with Google Drive via its internal browser API  list files, search,
  upload, download, create folders, move, rename, trash, share, and get file info.
  Use when the user wants to automate Google Drive, manage files in Google Drive,
  search Drive, upload to Drive, download from Drive, organize folders, share files,
  or perform any Google Drive file management task. Activate on mentions of
  Google Drive, Drive files, Drive folders, "my drive", shared drives, or
  Google Docs/Sheets/Slides file management.
allowed-tools: bash
---

# Google Drive

Control Google Drive in the browser via its internal v2 API. Requires the user to be logged into Google Drive in a browser tab.

## Prerequisites

The user must be logged into Google Drive in the browser. The skill uses page-context evaluation to call the Drive internal API with first-party auth (SAPISIDHASH).

If the Drive tab is not open, the script will open one automatically and wait for login.

## Commands

```bash
google-drive list [--folder=<id>] [--max=<n>]    # List files (default: root, 50 items)
google-drive search <query>                       # Search files by name/content
google-drive info <fileId>                        # Get file metadata
google-drive upload <local-path> [--folder=<id>] [--title=<name>]  # Upload a file
google-drive download <fileId> [--out=<path>]     # Download file content
google-drive mkdir <name> [--parent=<id>]         # Create a folder
google-drive move <fileId> --to=<folderId>        # Move a file
google-drive rename <fileId> --title=<name>       # Rename a file
google-drive trash <fileId>                       # Move to trash
google-drive untrash <fileId>                     # Restore from trash
google-drive share <fileId> --email=<addr> [--role=reader|writer|commenter]
google-drive unshare <fileId> --email=<addr>      # Remove sharing
```

## Architecture

- **Auth**: SAPISIDHASH generated from SAPISID cookie + SHA-1 hash, via `gapi.auth.getAuthHeaderValueForFirstParty([])`
- **API base**: `https://clients6.google.com/drive/v2internal/`
- **Upload endpoint**: `https://clients6.google.com/upload/drive/v2internal/files?uploadType=multipart`
- **API key**: Extracted from the Drive page's embedded scripts
- **Download**: Content fetched via same-origin redirect through `drive.google.com/uc?id=...&export=download` or read inline for text files

All operations run from the Google Drive tab's page context to leverage first-party cookies and auth.

## Notes

- Google Drive internal API uses v2internal, not the public v3 API
- Downloads of binary files generate a download URL; text content can be read inline
- Google Docs/Sheets/Slides use export links (PDF, docx, xlsx, etc.) rather than direct download
- The API key is session-specific; the script extracts it fresh each time
- Auth user index (0, 1, 2...) is detected from the current Drive URL
