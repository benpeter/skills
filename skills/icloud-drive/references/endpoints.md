# iCloud Drive Internal API Endpoints

## Server Prefix

The server prefix (e.g., `p54`) is session-specific. Extract it from the page's network requests via `performance.getEntriesByType('resource')`.

## Authentication

All requests use cookie-based auth (httpOnly cookies set by iCloud login):
- `credentials: 'include'`
- `Origin: https://www.icloud.com`
- **Important**: Use `Content-Type: text/plain` for POST requests to avoid CORS preflight

## Drive WS API (`{prefix}-drivews.icloud.com`)

### List Folder Contents
```
POST /retrieveItemDetailsInFolders
Body: [{drivewsid: "FOLDER::com.apple.CloudDocs::<uuid>", partialData: false}]
```
Returns folder metadata + `items[]` array with all children.

### Get Item Details (single file or folder)
```
POST /retrieveItemDetails
Body: {items: [{drivewsid: "FILE::com.apple.CloudDocs::<uuid>"}]}
```
Returns `{items: [...]}` with item metadata including etag.

### Rename
```
POST /renameItems
Body: {items: [{drivewsid: "<id>", name: "<new-name>", etag: "<etag>"}]}
```
Note: name is without extension. Extension can be set separately.

### Move
```
POST /moveItems
Body: {items: [{drivewsid: "<id>", destinationDrivewsId: "<folder-id>", etag: "<etag>", clientId: "<uuid>"}]}
```

### Trash
```
POST /moveItemsToTrash
Body: {items: [{drivewsid: "<id>", etag: "<etag>", clientId: "<uuid>"}]}
```
Etag is required. Obtain via retrieveItemDetails first.

## Doc WS API (`{prefix}-docws.icloud.com`)

### Lookup by Path
```
GET /ws/com.apple.CloudDocs/list/lookup_by_path?path=<path>&unified_format=true
```
Returns folder/file metadata for a path relative to root.

### Download (get pre-signed URL)
```
GET /ws/com.apple.CloudDocs/download/by_id?document_id=<docwsid>&unified_format=true
```
Returns `{data_token: {url: "https://cvws.icloud-content.com/..."}}`.
The content URL is pre-signed and can be fetched without auth.

### Upload (request upload slot)
```
POST /ws/com.apple.CloudDocs/upload/web
Body: {filename: "<name>", type: "FILE", content_type: "<mime>", size: <bytes>}
```
Returns `[{document_id: "<uuid>", url: "https://cws.icloud-content.com/..."}]`.

### Upload Content (to pre-signed URL)
```
POST <upload-url>
Content-Type: <mime-type>
Body: <file content>
```
Returns `{singleFile: {receipt, referenceChecksum, fileChecksum, size, wrappingKey}}`.

### Commit Upload
```
POST /ws/com.apple.CloudDocs/update/documents
Body: {
  command: "add_file",
  document_id: "<from upload>",
  path: {path: "<filename>", starting_document_id: "<parent-docwsid>"},
  btime: <timestamp>, mtime: <timestamp>,
  file_flags: {is_executable: false, is_hidden: false, is_writable: true},
  data: {receipt: "...", reference_signature: "...", signature: "...", size: N, wrapping_key: "..."}
}
```

### Create Folder
```
POST /ws/com.apple.CloudDocs/update/documents
Body: {
  command: "add_folder",
  document_id: "<new-uuid>",
  path: {path: "<folder-name>", starting_document_id: "<parent-docwsid>"},
  btime: <timestamp>, mtime: <timestamp>
}
```

### Recent Files
```
GET /ws/_all_/list/enumerate/recentDocs?limit=50&unified_format=true
```

### Enumerate Folders (alternate)
```
GET /v1/enumerate/folders/<item_id>?limit=50
```
Note: This only returns folders/subfolders, not files.

## ID Formats

- **drivewsid**: `{TYPE}::com.apple.CloudDocs::{UUID}` (e.g., `FILE::com.apple.CloudDocs::ABC-123`)
- **docwsid**: Just the UUID part (e.g., `ABC-123`)
- **item_id**: Base64-encoded internal ID (e.g., `CMrbilkQACIQ...`)
- Root folder: `root` (docwsid) or `FOLDER::com.apple.CloudDocs::root` (drivewsid)

## Common Response Fields

```
drivewsid, docwsid, name, extension, type (FILE|FOLDER),
size, dateCreated, dateModified, dateChanged, etag,
parentId, item_id, zone, isChainedToParent,
fileCount, directChildrenCount, assetQuota
```
