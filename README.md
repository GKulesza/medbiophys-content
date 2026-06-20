# medbiophys-content

Public content repository for the [MedBioPhys](https://github.com/KuuuGR/MedBioPhys) Flutter app.

This repository holds **remote-updatable content**: news feeds, department history, resource links, staff bios, and study regulations. The app ships bundled copies for offline use; when remote content is enabled, it syncs from here via `manifest.json`.

**Repository:** https://github.com/GKulesza/medbiophys-content

**Raw manifest URL (production):**

```
https://raw.githubusercontent.com/GKulesza/medbiophys-content/main/manifest.json
```

---

## Purpose

| Domain | What it contains | Manifest-tracked |
|--------|------------------|------------------|
| **News** | Locale-specific `feed.json` + hero/detail images | Feed JSON only (see [News images](#news-images)) |
| **History** | Markdown history page per locale | Yes |
| **Resources** | JSON list of external links per locale | Yes |
| **Staff** | JSON bios and role text per locale | Yes |
| **Regulations** | Five markdown documents per locale | Yes |

Supported locales: `en`, `pl`, `es`, `eo`, `isv`.

The app resolves content in this order: **memory → disk cache → bundled assets → remote (when enabled)**.

---

## Folder structure

```
medbiophys-content/
├── manifest.json                 # Content index (paths, versions, SHA-256)
├── news/
│   ├── en/
│   │   ├── feed.json
│   │   └── images/               # PNG/JPG referenced from feed markdown
│   ├── pl/
│   ├── es/
│   ├── eo/
│   └── isv/
├── history/
│   └── {en,pl,es,eo,isv}/history.md
├── resources/
│   └── {en,pl,es,eo,isv}/resources.json
├── staff/
│   └── {en,pl,es,eo,isv}/staff.json
├── regulations/
│   └── {en,pl,es,eo,isv}/
│       ├── regulations.md
│       ├── safety.md
│       ├── assessment.md
│       ├── exam.md
│       └── syllabus.md
└── testing/                      # Optional; not listed in manifest
    └── news/en/feed.json         # Integration-test fixture only
```

Current content set: **`2026-spring-content-v3`** — **45 manifest entries** (5 news + 5 history + 5 resources + 5 staff + 25 regulations).

---

## Manifest entry conventions

| Domain | Entry `id` | File `path` |
|--------|------------|--------------|
| News | `news/{locale}` | `news/{locale}/feed.json` |
| History | `history/{locale}/history` | `history/{locale}/history.md` |
| Resources | `resources/{locale}` | `resources/{locale}/resources.json` |
| Staff | `staff/{locale}` | `staff/{locale}/staff.json` |
| Regulations | `regulations/{locale}/{doc}` | `regulations/{locale}/{doc}.md` |

Remote download URL for an entry:

```
{baseUrl}/{path}
```

Example (after publish):

```
https://raw.githubusercontent.com/GKulesza/medbiophys-content/main/news/en/feed.json
```

### News images

Images live beside each feed under `news/{locale}/images/`. Markdown in `feed.json` uses **paths relative to the locale folder**, e.g. `![MRI scan](images/mri_scan.png)`.

The app maps these to cache ids like `news/en/images/mri_scan.png`. Images are **not** individually listed in the current manifest; updated JSON syncs remotely, but **new or changed image files** require either:

1. A new app release with updated bundled assets, or  
2. Adding per-image manifest entries (future enhancement).

All image paths referenced in each `feed.json` must exist on disk before publishing.

---

## How to publish content

Publishing means committing changes to the **`main`** branch of this repository. GitHub serves files via `raw.githubusercontent.com`; there is no separate CDN step.

### Workflow (from the MedBioPhys app repository)

1. Edit content under `MedBioPhys/medbiophys-content/` (or edit directly in this repo once split).
2. Keep bundled assets in sync: `medbiophys_flutter/assets/content/` mirrors the same tree for offline fallback.
3. Regenerate the manifest (see below).
4. Validate locally:

   ```bash
   python3 scripts/validate_content.py
   ```

5. Copy or push the updated tree to **this** repository.
6. Ensure `manifest.json` → `baseUrl` points at this repo's `main` branch raw URL.
7. Commit, push to `main`, and verify raw URLs load in a browser.

### What to include in this repo

| Include | Exclude |
|---------|---------|
| `manifest.json` | `.gitkeep` placeholders |
| `news/`, `history/`, `resources/`, `staff/`, `regulations/` | Translation caches, build artifacts |
| `testing/` (optional, documented as non-production) | App source code, Flutter assets path |

### Versioning

When shipping a coordinated content release:

1. Bump per-entry `version` strings in `manifest.json` for changed files.
2. Update top-level `contentSetId` (e.g. `2026-spring-content-v3`).
3. Set `publishedAt` to an ISO-8601 UTC timestamp.
4. Tag the commit if you want an immutable snapshot (optional).

---

## How to update `manifest.json`

**Do not edit SHA-256 hashes by hand.** Regenerate the manifest from on-disk files.

### Option A — App repo script (recommended)

From the MedBioPhys repository root:

```bash
python3 scripts/update_content_manifest.py
```

This script scans all manifest-tracked paths, computes SHA-256 and `sizeBytes`, and writes `medbiophys-content/manifest.json`.

> **Note:** The script currently hardcodes a staging `baseUrl`. Before publishing to this public repo, set `baseUrl` to:

```
https://raw.githubusercontent.com/GKulesza/medbiophys-content/main
```

(Parameterizing `baseUrl` in the script is planned for a later app phase.)

### Option B — Manual manifest fields

Each entry object:

```json
{
  "id": "news/en",
  "path": "news/en/feed.json",
  "version": "4",
  "sha256": "<64-char hex>",
  "sizeBytes": 13919,
  "locale": "en"
}
```

Top-level manifest fields:

| Field | Purpose |
|-------|---------|
| `schemaVersion` | Manifest format (currently `1`) |
| `contentSetId` | Human-readable release id |
| `publishedAt` | ISO-8601 UTC publication time |
| `minAppVersion` | Minimum app version required |
| `baseUrl` | Prefix for all entry downloads |
| `entries` | Array of content files |

---

## How to update SHA-256 hashes

### Single file (verify or compute)

```bash
shasum -a 256 news/en/feed.json
# macOS/Linux — output: "<hash>  news/en/feed.json"
```

Use only the 64-character hex digest (no spaces, lowercase preferred).

### All manifest entries

```bash
python3 scripts/update_content_manifest.py
python3 scripts/validate_content.py
```

`validate_content.py` checks that:

- Every manifest `path` exists on disk  
- On-disk SHA-256 matches manifest  
- News `feed.json` image references resolve to existing files  
- Bundled and remote copies stay aligned (when run from the app repo)

### After changing content

1. Edit the content file(s).
2. Bump the entry's `version` if the app should treat it as an update.
3. Regenerate manifest hashes.
4. Run validation.
5. Commit and push.

---

## Related documentation

In the MedBioPhys app repository:

- [`REMOTE_CONTENT_SETUP.md`](https://github.com/KuuuGR/MedBioPhys/blob/flutter-migration/docs/implementation/REMOTE_CONTENT_SETUP.md) — enabling remote sync, cache behaviour, staging vs production URLs  
- [`PUBLIC_REPOSITORY_MIGRATION_PLAN.md`](https://github.com/KuuuGR/MedBioPhys/blob/flutter-migration/docs/implementation/PUBLIC_REPOSITORY_MIGRATION_PLAN.md) — one-time migration from app repo to this public repository  
- `scripts/update_content_manifest.py`, `scripts/validate_content.py` — automation

---

## License

Content © MedBioPhys / Gdańsk University. See repository license file for terms of reuse.
