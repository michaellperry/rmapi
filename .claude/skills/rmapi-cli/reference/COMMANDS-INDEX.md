# Commands Reference Index

Complete command documentation organized by category. Select a category below to learn about specific commands.

## Command Categories

### [Navigation](navigation.md)
Move around the file tree and list contents.
- **pwd** - Print current working directory
- **cd** - Change directory  
- **ls** - List directory with multiple formatting options

### [File Operations](file-operations.md)
Upload, download, move, rename, and delete files and directories.
- **get** - Download single file
- **mget** - Download directory recursively
- **put** - Upload single file with options
- **mput** - Upload directory recursively
- **mv** - Move or rename files/directories
- **rm** - Delete files or directories
- **mkdir** - Create directories

### [Information](information.md)
View file metadata and account information.
- **stat** - Show detailed file metadata as JSON
- **account** - Display account and sync info
- **version** - Show rmapi version

### [Advanced](advanced.md)
Specialized operations for searching, annotations, and synchronization.
- **find** - Search files with regex and filtering
- **geta** - Generate annotated PDFs from documents
- **refresh** - Sync file tree with cloud
- **nuke** - Delete everything (dangerous!)

### [Global Flags and Offline](global-flags.md)
Global command flags and automation features.
- **-ni** - Non-interactive mode for scripts
- **-json** - JSON output for automation
- **version** - Offline version command
- **reset** - Offline reset command
- Environment variables and automation examples
