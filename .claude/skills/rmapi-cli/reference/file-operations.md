# File Operations

Commands for managing files and directories: downloading, uploading, moving, and deleting.

## Table of Contents

- [get](#get) - Download single file
- [mget](#mget) - Download directory
- [put](#put) - Upload file
- [mput](#mput) - Upload directory
- [mv](#mv) - Move or rename
- [rm](#rm) - Delete
- [mkdir](#mkdir) - Create directory

---

## get

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

## mget

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

## put

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

## mput

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

## mv

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

## rm

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

## mkdir

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
