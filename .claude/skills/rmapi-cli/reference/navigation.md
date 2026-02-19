# Navigation Commands

Commands for moving around and listing contents in the ReMarkable file tree.

## pwd

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

## cd

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

## ls

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
