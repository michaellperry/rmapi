# Information Commands

Commands for viewing file metadata and account information.

## stat

Display detailed metadata for a file or directory.

```bash
stat <entry>
```

**Arguments:**
- `<entry>` - File or directory path

**Output Format:**
JSON object containing full document metadata:
- `id` - Unique document ID
- `name` - Document name
- `type` - Document type (DocumentType, CollectionType, etc.)
- `parent` - Parent directory ID
- `lastModified` - Last modification timestamp
- `modifiedClient` - Client-side modification time
- `version` - Document version
- `pinned` - Whether document is pinned
- `starred` - Whether document is starred
- `metadatamodified` - Metadata modification timestamp
- `synced` - Sync status
- `bookmarked` - Whether document is bookmarked
- `tags` - Associated tags (array)

**Example:**
```bash
[/Documents]> stat my-document.pdf
{
  "id": "a1b2c3d4e5f6",
  "name": "my-document",
  "type": "DocumentType",
  "parent": "root",
  "lastModified": "2024-02-19T10:23:45.123Z",
  "modifiedClient": "2024-02-19T10:23:45.123Z",
  "version": 1,
  "pinned": false,
  "starred": true,
  "synced": true,
  "tags": ["work", "important"]
}
```

**Tab Completion:** Supports file/directory name completion

---

## account

Display current account information.

```bash
account
```

**Output:**
```
User: <email>, SyncVersion: <version>
```

Shows:
- **User** - Email address of authenticated user
- **SyncVersion** - ReMarkable API sync version in use (e.g., sync15)

**Example:**
```
[/]> account
User: user@example.com, SyncVersion: sync15
```

---

## version

Display the rmapi application version.

```bash
version
```

**Output:**
```
rmapi version: <version-string>
```

**Example:**
```
[/]> version
rmapi version: 0.0.14
```

**Note:** This is the interactive shell `version` command. For the offline version command (no auth required), use `rmapi version` outside the shell.
