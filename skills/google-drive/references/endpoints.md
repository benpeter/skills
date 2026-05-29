# Google Drive Internal API Endpoints

## Base URL

`https://clients6.google.com/drive/v2internal`

## Authentication

All requests require:
- `Authorization: SAPISIDHASH <timestamp>_<sha1hash>`  generated via `gapi.auth.getAuthHeaderValueForFirstParty([])`
- `X-Goog-AuthUser: <n>`  the user index from URL `/u/<n>/`
- `credentials: 'include'`  sends first-party cookies
- `key=<apiKey>`  API key from the Drive page's performance entries

## Endpoints

### List Files
```
GET /files?q=<query>&maxResults=<n>&fields=<fields>&orderBy=<order>&key=<key>
```

Query examples:
- `trashed=false and '<folderId>' in parents`
- `title contains 'search term'`
- `fullText contains 'content search'`
- `mimeType='application/vnd.google-apps.folder'`

### Get File Metadata
```
GET /files/<fileId>?fields=<fields>&key=<key>
```

### Create File/Folder (metadata only)
```
POST /files?key=<key>
Content-Type: application/json

{"title": "name", "mimeType": "...", "parents": [{"id": "root"}]}
```

### Upload File (multipart)
```
POST https://clients6.google.com/upload/drive/v2internal/files?uploadType=multipart&key=<key>
Content-Type: multipart/related; boundary="<boundary>"

--<boundary>
Content-Type: application/json; charset=UTF-8

{"title": "filename", "parents": [{"id": "root"}]}
--<boundary>
Content-Type: <mimeType>

<file content>
--<boundary>--
```

### Update File Metadata
```
PATCH /files/<fileId>?key=<key>
Content-Type: application/json

{"title": "new name"}
```

### Move File
```
PATCH /files/<fileId>?addParents=<newFolderId>&removeParents=<oldFolderId>&key=<key>
```

### Trash / Untrash
```
POST /files/<fileId>/trash?key=<key>
POST /files/<fileId>/untrash?key=<key>
```

### Permissions
```
GET /files/<fileId>/permissions?fields=items(id,emailAddress,role)&key=<key>
POST /files/<fileId>/permissions?sendNotificationEmails=false&key=<key>
DELETE /files/<fileId>/permissions/<permId>?key=<key>
```

### Download (binary files)
Downloads go through `https://drive.google.com/uc?id=<fileId>&export=download`.
Note: `alt=media` on `clients6.google.com` returns "Request unsafe for trusted domain"  this is blocked by Google's security layer.

### Export (Google Docs)
Use the `exportLinks` from file metadata, which point to `https://docs.google.com/feeds/download/...` or similar.

## Common Fields

```
id, title, mimeType, modifiedDate, createdDate, fileSize, 
owners(displayName,emailAddress), shared, parents(id),
webContentLink, webViewLink, exportLinks,
capabilities(canCopy,canDownload,canEdit,canDelete,canShare,canTrash,canRename)
```

## API Key Discovery

The API key is NOT hardcoded  it's extracted at runtime from the page's network requests via `performance.getEntriesByType('resource')`. This ensures the correct project key is always used, even if Google rotates keys.

Fallback key: `AIzaSyD_InbmSFufIEps5UAt2NmB_3LvBH3Sz_8`
