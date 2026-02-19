# ReMarkable Cloud API Research & Resources

## Overview
The ReMarkable Cloud API allows programmatic access to manage files on ReMarkable tablets. The API has evolved through multiple sync protocol versions (1.0, 1.5, 2.0+), with version 1.0 now deprecated (returning HTTP 410).

---

## Official & Community Documentation

### 1. **SplitBrain's ReMarkable API Wiki** (Primary Reference)
**URL:** https://github.com/splitbrain/ReMarkableAPI/wiki  
**Status:** Archived (Dec 2025) but still valuable  
**Coverage:** Comprehensive unofficial API documentation

**Key Pages:**
- **Authentication:** https://github.com/splitbrain/ReMarkableAPI/wiki/Authentication
  - Device registration via 8-letter code from https://my.remarkable.com/device/desktop/connect
  - Token refresh mechanism
  - Bearer token-based auth
  
- **Service Discovery:** https://github.com/splitbrain/ReMarkableAPI/wiki/Service-Discovery
  - Dynamic endpoint discovery
  - Storage and notification service URLs
  
- **Storage (Sync 1.0 API):** https://github.com/splitbrain/ReMarkableAPI/wiki/Storage
  - Document/folder metadata structure
  - CRUD operations for files
  - Upload/download via signed URLs
  - **Note:** This documents the OLD sync 1.0 protocol (now deprecated with HTTP 410)

---

## Known API Endpoints

### Authentication Endpoints
```
POST https://webapp.cloud.remarkable.com/token/json/2/device/new
POST https://my.remarkable.com/token/json/2/user/new  (token refresh)
```

### Sync 1.0 Endpoints (DEPRECATED - Returns HTTP 410)
```
Host: document-storage-production-dot-remarkable-production.appspot.com

GET  /document-storage/json/2/docs
PUT  /document-storage/json/2/upload/request
PUT  /document-storage/json/2/upload/update-status
PUT  /document-storage/json/2/delete
```

### Sync 1.5+ Endpoints (Current)
```
Host: https://internal.cloud.remarkable.com

POST /sync/v2/signed-urls/uploads     (get upload URLs for blobs)
POST /sync/v2/signed-urls/downloads   (get download URLs for blobs)
POST /sync/v2/sync-complete           (finalize sync operation)
```

**Important:** Sync 1.5+ uses a fundamentally different architecture based on:
- Hash-based content addressing
- Root index tracking with generation numbers
- Blob storage with signed Google Cloud Storage URLs
- Local cache at `~/.cache/rmapi/.tree` (or `~/Library/Caches/rmapi/.tree` on Mac)

---

## Related Projects & Implementations

### 1. **rmapi** (This Repository)
**URL:** https://github.com/juruen/rmapi  
**Status:** Archived (July 2024), unmaintained  
**Language:** Go  
**Features:**
- CLI shell for file management
- Sync 1.0 and 1.5 (experimental) support
- Non-interactive command execution
- See code at: `api/sync10/` and `api/sync15/`

**Key Implementation Files:**
- `api/sync15/blobstorage.go` - Blob storage operations
- `api/sync15/tree.go` - Hash tree sync implementation
- `api/sync15/remotestorage.go` - Remote storage interface
- `config/url.go` - API endpoint configuration

### 2. **rmfakecloud** by ddvk
**URL:** https://github.com/ddvk/rmfakecloud  
**Status:** Active (Latest: v0.0.27, Dec 2025)  
**Language:** Go  
**Purpose:** Self-hosted ReMarkable cloud replacement

**Features:**
- Supports Sync 1.0, 1.5, 2, 3, 4
- Compatible with reMarkable Paper Pro & Paper Pro Move
- Tested up to reMarkable software 3.25.1
- Email document sending
- Handwriting recognition
- Screen sharing (with TLS)
- WebDAV/FTP integrations

**Why It's Valuable:**
- Active reverse engineering of latest protocols
- Reference implementation of sync protocols
- Community knowledge base

**Documentation:** https://ddvk.github.io/rmfakecloud/

### 3. **ReMarkable Wiki**
**URL:** https://remarkablewiki.com/  
**Coverage:** Device internals, file formats, filesystem structure
- `.rm` file format for drawings
- `.metadata` file structure
- Device filesystem layout

---

## Technical Details

### Authentication Flow
1. Generate one-time 8-letter code at https://my.remarkable.com/device/desktop/connect
2. POST to `/token/json/2/device/new` with:
   - `code`: 8-letter code
   - `deviceDesc`: device type (e.g., "desktop-linux", "desktop-windows")
   - `deviceID`: UUID-4
3. Receive device token (JWT)
4. Exchange device token for user token via `/token/json/2/user/new`
5. Use user token (Bearer auth) for all subsequent API calls

### Token Structure (JWT)
User tokens contain:
- `auth0-profile.Email`: User email
- `scopes`: Determines sync version
  - `sync:fox`, `sync:tortoise`, `sync:hare` → Sync 1.5
  - Otherwise → Defaults to Sync 1.5 (since 1.0 is deprecated)
- `exp`: Expiration timestamp

**In rmapi code:** See `api/api.go` - `ParseToken()` function

### Sync 1.5 Protocol Architecture

**Core Concepts:**
1. **Hash Tree:** Documents organized in a content-addressable hash tree
2. **Root Index:** Single entry point with generation number for optimistic locking
3. **Blob Storage:** Files stored as blobs identified by SHA-256 hashes
4. **Entries:** Each document/folder represented as an entry with hash, type, ID

**Sync Flow:**
1. Fetch root index hash and generation number
2. Download hash tree structure using blob hashes
3. Compare local cache with remote state
4. Upload/download changed blobs
5. Update root index with new generation number
6. Call sync-complete endpoint

**Cache Location:**
- Linux/Windows: `~/.cache/rmapi/.tree` or `%LocalAppData%\rmapi\.tree`
- macOS: `~/Library/Caches/rmapi/.tree`

**Entry Format:**
```
{hash}:80:{documentID}:0:{size}
```
- `hash`: SHA-256 of content
- `80`: Type (file=80)
- `documentID`: UUID
- `size`: File size in bytes

---

## File Format References

### Document ZIP Structure
When downloading, you get a ZIP containing:
- `{id}.content` - Document metadata (JSON)
- `{id}.pagedata` - Page-specific data
- `{id}.lines` - Drawing/annotation data (if present)
- `{id}.pdf` - Original PDF (for uploaded documents)
- `{id}.thumbnails/*.jpg` - Page thumbnails

### Drawing Format (.rm files)
**Reference:** https://plasma.ninja/blog/devices/remarkable/binary/format/2017/12/26/reMarkable-lines-file-format.html  
**Also:** https://github.com/ax3l/lines-are-beautiful

Binary format containing:
- Header with version info
- Layer data
- Stroke information (points, pressure, pen type)

---

## Why rmapi Fails on Windows (But Works on Mac)

### Root Cause Analysis:

1. **Mac has cached tree file** - If Mac ran successfully before, it has `~/Library/Caches/rmapi/.tree` with the document structure, avoiding API calls
2. **Windows has no cache** - Fresh sync attempt hits the deprecated Sync 1.0 endpoints
3. **API Rollout** - Sync 1.0 endpoints return HTTP 410 (Gone), forcing Sync 1.5 usage
4. **Version Detection** - rmapi defaults to Sync 1.5 but may have cached auth from different protocol versions

**See code:**
- `api/api.go:76` - "Default to Sync 1.5 as Sync 1.0 is deprecated (HTTP 410)"
- `api/sync15/common.go` - Cache file management
- `api/sync15/blobstorage.go` - Sync 1.5 implementation

---

## Current State & Migration Path

### Sync Protocol Timeline:
- **Sync 1.0** (document-storage API) → Deprecated, returns HTTP 410
- **Sync 1.5** (hash tree + blob storage) → Current in rmapi
- **Sync 2.0, 3.0, 4.0** → Newer versions supported by rmfakecloud

### For Working Implementation:
1. **Use rmfakecloud** - Self-host or contribute to active project
2. **Study ddvk's implementation** - Most up-to-date reverse engineering
3. **Check token scopes** - Ensure `sync:fox`/`sync:tortoise`/`sync:hare` present
4. **Verify endpoints** - Use `https://internal.cloud.remarkable.com` for sync 1.5+

---

## Additional Resources

### GitHub Discussions
- **rmapi Archive Discussion:** https://github.com/juruen/rmapi/discussions/313
- **API v2 Design Discussion:** https://github.com/juruen/rmapi/issues/54
  - Good architectural discussion about high-level vs low-level API design
  - References to good Go API client patterns

### Code References
- **morningpaper2remarkable** by @jessfraz: https://github.com/jessfraz/morningpaper2remarkable
  - Example external usage of rmapi as library

### ReMarkable Community
- ReMarkable subreddit: r/RemarkableTablet
- ReMarkable Discord communities

---

## Environment Variables (rmapi)

```bash
RMAPI_CONFIG      # Custom config file path (default: ~/.rmapi or ~/.config/rmapi/rmapi.conf)
RMAPI_TRACE=1     # Enable trace logging
RMAPI_AUTH        # Override auth endpoint
RMAPI_DOC         # Override document storage endpoint  
RMAPI_HOST        # Override all endpoints
RMAPI_CONCURRENT  # Max concurrent requests for sync15 (default: 20)
RMAPI_USE_HIDDEN_FILES=1  # Include hidden files/folders
```

---

## Legal Disclaimer

**From SplitBrain Wiki:**
> Use of this API is on your own risk. Do NOT contact the reMarkable support team if anything doesn't work as described here. This project is not affiliated with reMarkable AS, Oslo.

All API usage is reverse-engineered and unofficial. ReMarkable AS provides no official public API documentation.

---

## Next Steps for Research

1. **Study rmfakecloud source** - Most current protocol implementation
2. **Compare sync versions** - Understand differences between 1.5, 2.0, 3.0, 4.0
3. **Protocol sniffing** - Use mitmproxy or similar to capture official client traffic
4. **Check reMarkable software updates** - API may change with device firmware updates

---

**Document Created:** February 18, 2026  
**Based on Analysis of:** rmapi codebase, SplitBrain wiki, rmfakecloud project
