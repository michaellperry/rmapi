# rmapi Commands Reference

Complete detailed documentation of all rmapi commands with flags, options, and examples.

## Table of Contents

### Navigation Commands
- [pwd](#pwd) - Print current directory
- [cd](#cd) - Change directory
- [ls](#ls) - List directory

### File Operations
- [get](#get) - Download single file
- [mget](#mget) - Download directory recursively
- [put](#put) - Upload file
- [mput](#mput) - Upload directory recursively
- [mv](#mv) - Move or rename
- [rm](#rm) - Delete

### Directory
- [mkdir](#mkdir) - Create directory

### Information
- [stat](#stat) - File metadata
- [account](#account) - Account info
- [version](#version) - Version info

### Advanced
- [find](#find) - Search files
- [geta](#geta) - Annotated PDFs
- [refresh](#refresh) - Sync with remote
- [nuke](#nuke) - Delete everything

---

## Navigation Commands

### pwd

Print the current working directory path.

```bash
pwd
```

**Arguments:** None

**Output:** Current path in the ReMarkable file tree

**Example:**
```
[/Documents]> pwd
/Documents
```

---

### cd

Change the current working directory.

```bash
cd <directory>
```

**Arguments:**
- `<directory>` - Target directory path (relative or absolute)

**Error Handling:**
- Fails if directory doesn't exist
- Fails if path points to a file instead of directory

**Example:**
```
[/]> cd Documents
[/Documents]> pwd
/Documents
```

**Tab Completion:** Supports directory name completion

---

### ls

List directory contents with various formatting and sorting options.

```bash
ls [options] [path]
```

**Options:**

| Flag | Short | Long | Default | Description |
|------|-------|------|---------|-------------|
| `-c` | | `--compact` | false | Display in compact format |
| `-l` | | `--long` | false | Display in long format (with timestamps) |
| `-r` | | `--reverse` | false | Reverse sort order |
| `-d` | | `--group-directories` | false | Group directories first |
| `-t` | | `--time` | false | Sort by modification time instead of name |
| `-s` | | `--show-templates` | false | Include template files (hidden by default) |

**Arguments:**
- `[path]` - Optional directory to list (defaults to current directory)

**Output Formats:**

Default:
```
[d]    folder-name
[f]    document.pdf
```

Compact (`-c`):
```
folder-name/
document.pdf
```

Long (`-l`):
```
2024-02-19T10:23:45.123Z document.pdf/
2024-01-15T09:12:34.567Z folder-name
```

JSON (`-json` flag):
```json
[
  {
    "id": "...",
    "name": "folder-name",
    "type": "CollectionType",
    ...
  }
]
```

**Examples:**
```bash
# List current directory
ls

# List with timestamps sorted by time
ls -l -t

# List specific path in compact format
ls -c /Documents

# Group directories first, reverse sort
ls -d -r

# Show templates that are normally hidden
ls -s
```

---

## File Operations

### get

Download a single file from the cloud to the local system.

```bash
get <remote_file>
```

**Arguments:**
- `<remote_file>` - Path to file in ReMarkable cloud

**Behavior:**
- Downloads file to current working directory
- Output filename is `<name>.rmdoc`
- Shows download progress

**Error Handling:**
- Fails if file doesn't exist
- Fails if path is a directory

**Example:**
```
[/Documents]> get papers/research.pdf
downloading: [papers/research.pdf]...
OK
# Creates: research.pdf.rmdoc
```

**Tab Completion:** Supports file name completion

---

### mget

Recursively download a directory and all its contents.

```bash
mget [options] <remote_dir>
```

**Options:**

| Flag | Short | Description |
|------|-------|-------------|
| `-i` | | Incremental sync (only update changed files) |
| `-o` | | Output folder (default: `.`) |
| `-d` | | Remove deleted/moved files from local copy |

**Arguments:**
- `<remote_dir>` - Directory path to download

**Behavior:**
- Creates local directory structure matching remote
- Files saved with `.rmdoc` extension
- Preserves modification times
- Creates directories as needed

**Incremental Mode (`-i`):**
- Only downloads files if remote is newer than local
- Skips unchanged files
- Uses file modification times for comparison

**Remove Deleted Mode (`-d`):**
- Removes files/folders locally if they were deleted remotely
- Requires explicit output folder (can't use `-d` with `-o .`)
- Useful for maintaining synchronized backup

**Examples:**
```bash
# Download entire library to local backup
mget -o ./backup /

# Incremental sync (updates only changed files)
mget -i -o ./Documents /Documents

# Sync and remove locally deleted remote files
mget -d -o ./sync-folder /Documents

# Download single directory
mget -o ./papers /Documents/Papers
```

**Tab Completion:** Supports directory name completion

---

### put

Upload a single file to the cloud.

```bash
put [options] <local_file> [remote_dir]
```

**Options:**

| Flag | Description |
|------|-------------|
| `--force` | Overwrite existing file (delete and recreate) |
| `--content-only` | Replace PDF content preserving metadata |
| `--coverpage=<0\|1>` | Set/unset coverpage (0=disable, 1=set first page as cover) |

**Arguments:**
- `<local_file>` - Local file path to upload
- `[remote_dir]` - Optional remote directory (defaults to current directory)

**Behavior:**
- Uploads file to remote cloud storage
- Document metadata stored in cloud
- File extension determines document type

**Conflict Resolution:**

| Scenario | Behavior |
|----------|----------|
| File already exists | Error (use `--force` or `--content-only`) |
| With `--force` | Delete existing document, upload new one |
| With `--content-only` | Replace PDF content, preserve metadata |

**PDF Content Mode (`--content-only`):**
- Only works with PDF files
- Preserves document metadata, notes, and properties
- Updates the PDF content while keeping document information
- Useful when updating content of important documents

**Coverpage Option:**
- `--coverpage=1` - Sets first page as document cover
- `--coverpage=0` - Disables coverpage (default)
- Only works with PDF uploads

**Examples:**
```bash
# Simple upload to current directory
put ~/Documents/paper.pdf

# Upload to specific remote directory
put ~/Downloads/article.pdf /Documents

# Force overwrite existing document
put --force ~/new-version.pdf /Documents

# Replace PDF content only (keep metadata)
put --content-only ~/updated-version.pdf /Documents

# Upload with first page as cover
put --coverpage=1 ~/fancy-document.pdf

# All options combined
put --force --coverpage=1 ~/document.pdf /Documents
```

**Tab Completion:** Supports local file system completion

---

### mput

Recursively upload a local directory and all its contents.

```bash
mput [options] <remote_dir>
```

**Options:**

| Flag | Description |
|------|-------------|
| `--src` | Source directory (default: `.` - current directory) |

**Arguments:**
- `<remote_dir>` - Destination directory in cloud

**Behavior:**
- Creates remote directory structure matching local
- Skips files that already exist in remote
- Skips non-supported file types
- Preserves modification times
- Shows tree-formatted progress output

**File Support:**
- Supports: PDF, EPUB, and other document formats
- Skips: Hidden files (unless `RMAPI_USE_HIDDEN_FILES=1`), unsupported types

**Example Output:**
```
/Documents
├── creating directory [subfolder]... complete
├── uploading: [document.pdf]... complete
│   ├── document2.pdf already exists
│   └── article.epub already exists
└── downloading... complete
```

**Examples:**
```bash
# Upload current directory to remote location
mput /Documents

# Upload specific local folder
mput --src ~/my-docs /Documents/imports

# Upload nested structure
mput --src ~/backups/2024 /Archives/2024
```

**Tab Completion:** Supports directory name completion

---

### mv

Move or rename a file or directory.

```bash
mv <source> <destination>
```

**Arguments:**
- `<source>` - File or directory to move/rename
- `<destination>` - Target path or new name

**Modes:**

**Move to Directory:**
- If destination is a directory, source is moved into it
- Source keeps original name
- Works with multiple source files

**Rename:**
- If destination is a path in same directory, source is renamed
- Can rename individual files
- Cannot rename multiple files at once

**Behavior:**
- Prevents circular moves (moving directory into itself)
- Updates file tree after operation
- Syncs changes with cloud

**Error Handling:**
- Fails if destination is a file
- Fails if trying to move directory into itself
- Warns if renaming multiple files (only renames first)

**Examples:**
```bash
# Rename file
mv old-name.pdf new-name.pdf

# Move to another directory
mv document.pdf /Archive

# Move multiple files to directory
mv file1.pdf file2.pdf /Documents

# Move directory with contents
mv /OldFolder /Archive/OldFolder

# Rename directory
mv /Documents /MyDocuments
```

**Tab Completion:** Supports file/directory name completion

---

### rm

Delete files or directories from the cloud.

```bash
rm [options] <entry>...
```

**Options:**

| Flag | Description |
|------|-------------|
| `-r`, `--recursive` | Delete non-empty directories (required for folders with contents) |

**Arguments:**
- `<entry>...` - One or more files or directories to delete

**Behavior:**
- Deletes files immediately
- Requires `-r` flag for directories with contents
- Can delete multiple entries in one command
- Syncs changes with cloud

**Examples:**
```bash
# Delete single file
rm old-document.pdf

# Delete multiple files
rm file1.pdf file2.pdf file3.pdf

# Delete empty directory
rm empty-folder

# Delete directory with contents
rm -r /Archive

# Delete multiple entries
rm document.pdf folder/ another-file.pdf
```

**Tab Completion:** Supports file/directory name completion

---

## Directory Commands

### mkdir

Create a new directory in the cloud.

```bash
mkdir <directory>
```

**Arguments:**
- `<directory>` - Path to new directory

**Behavior:**
- Creates single directory
- Fails if parent directory doesn't exist
- Fails if directory already exists

**Examples:**
```bash
# Create directory in current location
mkdir new-folder

# Create directory with path
mkdir /Documents/new-subfolder
```

**Tab Completion:** Supports directory name completion

---

## Information Commands

### stat

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

### account

Display current account information.

```bash
account
```

**Output:**
```
User: <email>, SyncVersion: <version>
```

**Example:**
```
[/]> account
User: user@example.com, SyncVersion: sync15
```

---

### version

Display the rmapi application version.

```bash
version
```

**Output:**
```
rmapi version: <version-string>
```

---

## Advanced Commands

### find

Recursively search for files matching criteria.

```bash
find [options] [directory] [regexp]
```

**Options:**

| Flag | Description |
|------|-------------|
| `-c`, `--compact` | Compact output format (no type prefix) |
| `--tag` | Filter by tag (can specify multiple times for OR matching) |
| `--starred` | Only show starred/favorited files |

**Arguments:**
- `[directory]` - Start directory (defaults to current directory)
- `[regexp]` - Regular expression pattern for filename matching

**Behavior:**
- Recursively searches entire subtree
- Returns both files and directories
- Displays full path from search root
- Supports regex patterns in filenames

**Output Formats:**

Default:
```
[d] /path/to/folder/
[f] /path/to/file.pdf
```

Compact (`-c`):
```
/path/to/folder/
/path/to/file.pdf
```

JSON:
```json
[
  {
    "id": "...",
    "name": "file.pdf",
    ...
  }
]
```

**Tag Filtering:**
- Multiple `--tag` flags use OR logic
- Matches files with ANY listed tag
- Only matches documents with tags field

**Examples:**
```bash
# Find all PDFs
find / "\.pdf$"

# Find documents modified in Q1 2024
find -c / "2024-0[123]"

# Find documents with 'work' tag
find --tag work /

# Find documents with either 'work' or 'urgent' tag
find --tag work --tag urgent /

# Find starred documents
find --starred /Documents

# Find starred items with 'important' tag
find --starred --tag important /

# Find in specific directory
find /Documents pdf

# Complex pattern with case-insensitive matching
find -c / "[0-9]{4}-[0-9]{2}" 
```

**Tab Completion:** Supports directory name completion

---

### geta

Download a file and generate a PDF with all annotations.

```bash
geta [options] <remote_file>
```

**Options:**

| Flag | Description |
|------|-------------|
| `-p` | Add page numbers to output PDF |
| `-a` | Include all pages (not just annotated ones) |
| `-n` | Annotations only (exclude original document content) |

**Behavior:**
- Downloads document as ZIP file
- Processes annotations from document metadata
- Generates new PDF: `<name>-annotations.pdf`
- Page numbers optional for reference

**Output:**
- Original ZIP: `<name>.zip`
- Generated PDF: `<name>-annotations.pdf`

**Annotation Modes:**

| Option | Result |
|--------|--------|
| None | Annotated pages only |
| `-a` | All pages (annotated + blank) |
| `-n` | Annotations as PDF only |
| `-a -n` | All page annotations in PDF |

**Page Numbers:**
- `-p` adds page numbers to generated PDF
- Useful for referencing annotated content

**Examples:**
```bash
# Generate PDF with annotations
geta /Documents/research.pdf
# Creates: research-annotations.pdf

# Include page numbers in output
geta -p /Documents/notes.pdf

# Get all pages with annotations
geta -a /Documents/book.pdf

# Get annotations only (no original content)
geta -n /Documents/article.pdf

# Combine options: all pages with page numbers
geta -a -p /Documents/textbook.pdf
```

**Tab Completion:** Supports file name completion

---

### refresh

Synchronize the local file tree with remote cloud changes.

```bash
refresh
```

**Output:**
```
root hash: <hash>
generation: <number>
```

**Behavior:**
- Fetches latest file tree from cloud
- Updates local cache
- Detects remotely deleted files
- Updates current directory context

**Error Handling:**
- If current directory was deleted remotely, moves to root
- Notifies if current path becomes invalid

**Use Cases:**
- Sync after using web interface
- Update for changes made on other devices
- Get latest document list

**Example:**
```
[/Documents]> refresh
root hash: a1b2c3d4e5f6g7h8i9j0
generation: 42
```

---

### nuke

**DESTRUCTIVE OPERATION** - Delete all files and folders from account.

```bash
nuke
```

**Safety:**
- Requires confirmation by typing: `YES`
- Case-sensitive confirmation
- Cannot be undone

**Behavior:**
- Deletes everything in ReMarkable account
- Clears local file tree representation
- Cannot be undone from command line

**⚠️ WARNING:**
```
This will DELETE EVERYTHING! type [YES]:
```

**Example:**
```
[/]> nuke
Are you sure, this will DELETE EVERYTHING! type [YES]: YES
Nuking
```

---

## Global Flags

All commands support these global flags when run outside shell:

### `-ni` Flag (Non-Interactive)

Prevents authentication prompts and interactive input.

```bash
rmapi -ni <command> [args]
```

**Use Cases:**
- Scripts and automation
- Headless servers
- CI/CD pipelines

**Behavior:**
- Skips all interactive dialogs
- Uses existing stored authentication
- Fails if no authentication available

**Example:**
```bash
# Backup in non-interactive mode
rmapi -ni mget -o ./backup /

# Upload without interaction
rmapi -ni put ~/file.pdf /Documents
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
```json
{
  "command": "ls",
  "results": [...]
}
```

**Example:**
```bash
# Get directory listing as JSON
rmapi -json ls /Documents

# Parse with jq
rmapi -json ls / | jq '.[] | select(.type == "DocumentType")'
```

