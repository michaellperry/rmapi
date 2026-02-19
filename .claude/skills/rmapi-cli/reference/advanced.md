# Advanced Commands

Specialized commands for searching, generating annotated PDFs, syncing, and account management.

## find

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

## geta

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

## refresh

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

## nuke

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
Are you sure, this will DELETE EVERYTHING! type [YES]:
```

**Example:**
```
[/]> nuke
Are you sure, this will DELETE EVERYTHING! type [YES]: YES
Nuking
```

---

## Related: Offline Commands

Use outside the shell without authentication:

```bash
rmapi version              # Show version
rmapi reset                # Clear authentication
```

See [Global Flags](global-flags.md) for more information about running commands non-interactively and with JSON output.
