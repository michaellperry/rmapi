# Global Flags and Non-Interactive Usage

Documentation for global flags that apply to all rmapi commands.

## Global Flags

All commands support these global flags when run outside the shell:

### `-ni` Flag (Non-Interactive)

Prevents authentication prompts and interactive input.

```bash
rmapi -ni <command> [args]
```

**Use Cases:**
- Scripts and automation
- Headless servers
- CI/CD pipelines
- Scheduled backups
- Unattended operations

**Behavior:**
- Skips all interactive dialogs
- Uses existing stored authentication
- Fails with error if no authentication available
- Cannot use interactive features

**Example:**
```bash
# Backup in non-interactive mode
rmapi -ni mget -o ./backup /

# Upload without interaction
rmapi -ni put ~/file.pdf /Documents

# Delete in script context
rmapi -ni rm old-file.pdf

# Refresh tree in cron job
rmapi -ni refresh
```

### `-json` Flag (JSON Output)

Outputs results in JSON format instead of human-readable text.

```bash
rmapi -json <command> [args]
```

**Supported Commands:**
- `ls` - Directory listing as JSON
- `stat` - File metadata as JSON
- `find` - Search results as JSON

**Output Format:**
Each supported command returns structured JSON output.

**Shell vs Non-Interactive:**
- Use `-json` to get structured output in non-interactive shell commands
- Works with `rmapi -json <command>`
- Can be combined with `-ni` for fully automated workflows

**Example - List as JSON:**
```bash
rmapi -json ls /Documents
```

Output:
```json
[
  {
    "id": "a1b2c3d4e5f6",
    "name": "document.pdf",
    "type": "DocumentType",
    "parent": "root-id",
    "starred": true,
    "tags": ["work"]
  },
  {
    "id": "b2c3d4e5f6g7",
    "name": "folder",
    "type": "CollectionType",
    "parent": "root-id",
    "starred": false,
    "tags": []
  }
]
```

**Example - File Metadata as JSON:**
```bash
rmapi -json stat /Documents/file.pdf
```

Output:
```json
{
  "id": "a1b2c3d4e5f6",
  "name": "file",
  "type": "DocumentType",
  "parent": "parent-id",
  "lastModified": "2024-02-19T10:23:45.123Z",
  "modifiedClient": "2024-02-19T10:23:45.123Z",
  "version": 1,
  "pinned": false,
  "starred": true,
  "synced": true,
  "tags": ["important", "review"]
}
```

**Example - Search Results as JSON:**
```bash
rmapi -json find /Documents "\.pdf$"
```

**Parsing with jq:**

Filter documents only (exclude folders):
```bash
rmapi -json ls / | jq '.[] | select(.type == "DocumentType")'
```

Get all starred documents:
```bash
rmapi -json find / | jq '.[] | select(.starred == true)'
```

Extract names of all PDFs:
```bash
rmapi -json find / "\.pdf$" | jq -r '.[].name'
```

Find by tag:
```bash
rmapi -json find / | jq '.[] | select(.tags[] == "work")'
```

### Combining Flags

Flags can be combined for powerful workflows:

```bash
# Non-interactive JSON output for automation
rmapi -ni -json ls /Documents

# Script-safe backup with progress
rmapi -ni mget -o ./backup /

# Automated search with JSON parsing
rmapi -ni -json find /Documents "2024" | jq -r '.[].name'
```

---

## Offline Commands

Commands that don't require authentication (no shell needed):

### version

Print the rmapi application version.

```bash
rmapi version
```

**Output:**
```
<version-string>
```

**Example:**
```bash
$ rmapi version
0.0.14
```

---

### reset

Remove the local config file and clear authentication.

```bash
rmapi reset
```

**Behavior:**
- Deletes stored authentication token
- Removes local configuration
- Requires re-authentication on next use
- Safe operation with no other side effects

**Use Cases:**
- Switch to different account
- Clear authentication after compromise
- Reset application state
- Troubleshoot authentication issues

**Example:**
```bash
$ rmapi reset
$ rmapi  # Now prompts for authentication again
```

---

## Environment Variables

### RMAPI_USE_HIDDEN_FILES

Control whether hidden files (prefixed with `.`) are displayed.

```bash
# Show hidden files
export RMAPI_USE_HIDDEN_FILES=1
rmapi ls /

# Hide hidden files (default)
export RMAPI_USE_HIDDEN_FILES=0
rmapi ls /
```

**Values:**
- `0` or unset - Don't show hidden files (default)
- `1` - Show all files including those starting with `.`

**Affects Commands:**
- `ls` - Shows/hides hidden entries
- `mput` - Skips/includes hidden local files when uploading

**Example:**
```bash
# Show all files including hidden
RMAPI_USE_HIDDEN_FILES=1 rmapi ls

# Upload including hidden files
RMAPI_USE_HIDDEN_FILES=1 rmapi mput /Documents
```

---

## Automation Examples

### Scheduled Backup (Cron)

```bash
#!/bin/bash
# Incremental backup every night
0 2 * * * /usr/local/bin/rmapi -ni mget -i -o ~/rmapi-backup /
```

### Sync Script

```bash
#!/bin/bash
# Two-way sync
set -e

# Download with cleanup
rmapi -ni mget -d -o ./local-copy /remote-folder

# Upload new files
rmapi -ni mput --src ./local-changes /remote-folder

# Refresh to confirm
rmapi -ni refresh
```

### JSON Processing Pipeline

```bash
#!/bin/bash
# Find and process all work documents
rmapi -ni -json find / | jq -r '.[] | select(.tags[] == "work") | .name'

# Find all starred files and display
rmapi -ni -json find --starred / | jq -r '.[] | "\(.name) (starred)"'
```

### Unattended CI/CD Upload

```bash
#!/bin/bash
# Upload build artifacts (non-interactive, no prompts)
rmapi -ni put ./build/document.pdf /CI-Builds/$(date +%Y-%m-%d)
```

