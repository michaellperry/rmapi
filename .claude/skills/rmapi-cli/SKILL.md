---
name: rmapi-cli
description: Complete documentation of the rmapi command line interface, including all commands, options, flags, and use cases. Use when users need to interact with the ReMarkable Cloud API, perform document management operations, sync files, or understand rmapi CLI commands and their options.
---

# rmapi CLI Documentation

Complete reference for the rmapi command line interface - a Go application for programmatic access to the ReMarkable Cloud API.

## Quick Start

```bash
# Interactive shell (requires authentication)
rmapi

# Run single command
rmapi <command> [args]

# Global flags
rmapi -ni <command>          # Non-interactive (no auth prompts)
rmapi -json <command>         # JSON output format
```

## Shell Commands Overview

Once authenticated, rmapi opens an interactive shell with these command categories:

- **Navigation**: [pwd, cd, ls](reference/navigation.md) - Move around and list contents
- **File Operations**: [get, mget, put, mput, mv, rm, mkdir](reference/file-operations.md) - Upload, download, move, delete
- **Information**: [stat, account, version](reference/information.md) - View metadata and account info
- **Advanced**: [find, geta, refresh, nuke](reference/advanced.md) - Search, annotations, sync, and dangerous ops

## Global Flags and Offline Commands

- **Flags**: [-ni, -json, environment variables](reference/global-flags.md)
- **Offline Commands**: `version`, `reset` (no authentication needed)
- **Automation**: Non-interactive workflows and scripting examples

## File Format Support

rmapi works with various document formats:
- **PDF** (.pdf) - Portable Document Format
- **EPUB** (.epub) - E-book format
- **ReMarkable** (.rm) - Native ReMarkable format

The `put` command uploads with `zip` extension; `get` downloads with `.rmdoc` extension.

## Authentication

rmapi uses the ReMarkable Cloud API and requires authentication:
1. On first run, a browser window opens for authentication
2. The auth token is stored locally
3. Use `rmapi reset` to clear authentication and start over
4. Use `-ni` flag to skip authentication prompts in scripts

## Common Workflows

### Backup Remote Files
```bash
rmapi mget -o ./backup /
```

### Incremental Sync
```bash
rmapi mget -i -o ./local-copy /documents
```

### Search Documents
```bash
rmapi find -c / "2024.*"
```

### Find Starred Items
```bash
rmapi find --starred /
```

### Filter by Tags
```bash
rmapi find --tag work --tag important /
```

### Generate Annotated PDFs
```bash
rmapi geta -p "document"
# Generates: document-annotations.pdf
```

### Non-Interactive Backup (Cron/CI)
```bash
rmapi -ni mget -i -o ~/rmapi-backup /
```

### JSON Output for Scripting
```bash
rmapi -ni -json find /Documents | jq '.[] | select(.starred)'
```

