# wp-import

Import WordPress content from exported files to a WordPress site.

## Script Location

Scripts are in the shared `../../scripts/` folder relative to this skill.

## Setup (first time only)

```bash
cd ../../scripts && npm install
```

## Quick Start

```bash
cd ../../scripts && node wp-import.js --dry-run  # Preview first
cd ../../scripts && node wp-import.js            # Execute import
```

## CLI Options

| Option | Description | Default |
|--------|-------------|---------|
| `-u, --url <url>` | WordPress site URL | `WP_REMOTE_URL` env |
| `--user <user>` | WordPress username | `WP_REMOTE_USER` env |
| `--password <pass>` | Application password | `WP_REMOTE_APP_PASSWORD` env |
| `-i, --input <dir>` | Input directory | `./export` |
| `-t, --type <type>` | `posts`, `pages`, or `all` | `all` |
| `-m, --mode <mode>` | `create`, `update`, or `sync` | `sync` |
| `--no-media` | Skip uploading media files | - |
| `--no-plugins` | Skip importing plugin data | - |
| `--dry-run` | Show what would be imported | - |
| `-v, --verbose` | Verbose output | - |

## Import Modes

| Mode | Behavior |
|------|----------|
| `create` | Only create new items (skip if slug exists) |
| `update` | Only update existing items (skip if not found) |
| `sync` | Create new AND update existing (default) |

## Examples

```bash
# Always dry-run first!
cd ../../scripts && node wp-import.js --dry-run

# Import all content
cd ../../scripts && node wp-import.js

# Only update existing pages
cd ../../scripts && node wp-import.js --type pages --mode update

# Only create new posts
cd ../../scripts && node wp-import.js --type posts --mode create

# Import to different site
cd ../../scripts && node wp-import.js --url https://staging.example.com
```

## Import Process

### Per Content Item
1. Read `metadata.json` for slug, title, status
2. Check if post/page exists by slug on target site
3. Apply import mode (create/update/sync)
4. Upload media from `media/` folder to WordPress
5. Replace local `./media/` paths with new WordPress URLs
6. Create/update post via REST API
7. Import plugin meta via `all_meta` field

### Global Plugin Data
1. Read manifest.json for exported plugins list
2. Import Contact Form 7 forms (create/update via CF7 REST API)
3. Import plugin options via WordPress Settings API

## What Gets Imported

### From each `posts/{slug}/` or `pages/{slug}/`:
| File | Imported As |
|------|-------------|
| `body.html` | Post/page content |
| `metadata.json` | Title, status, slug, excerpt, categories, tags |
| `media/` | Re-uploaded, URLs replaced in content |
| `seo-by-rank-math.json` | Post meta via `all_meta` |
| `wordpress-seo.json` | Post meta via `all_meta` |
| Other `*.json` | Post meta via `all_meta` |

### From export root:
| File | Imported Via |
|------|--------------|
| `contact-form-7.json` | CF7 REST API |
| `seo-by-rank-math.json` | Settings API |
| Other plugin JSON | Settings API |

## Required for Full Meta Import

Install the MU-plugin on target WordPress:

```bash
cp export/_wordpress-plugin/expose-all-meta.php \
   /path/to/wordpress/wp-content/mu-plugins/
```

This enables writing to `all_meta` field via REST API.

## Script Files

Scripts are located in `../../scripts/`:
- `wp-import.js` - Main import script
- `lib/api-client.js` - WordPress REST API client
- `lib/file-utils.js` - File operations
- `lib/media-handler.js` - Media upload with URL mapping
- `config.js` - Configuration loading

---

## Character Encoding (UTF-8)

**CRITICAL**: The import scripts handle UTF-8 encoding correctly. All content with special characters (German umlauts: ä, ö, ü, ß, accented characters: é, è, ñ, etc.) will be imported correctly.

### Requirements
- Source files must be UTF-8 encoded
- Target WordPress must use UTF-8 charset (default for modern WordPress)
- REST API payloads are sent with `Content-Type: application/json; charset=utf-8`

### What Gets Imported
- `body.html` - Full UTF-8 Gutenberg content
- `metadata.json` - Titles, excerpts with special characters
- `*.json` plugin files - SEO meta with special characters

### Pre-Import Check
Verify your export files have correct encoding:
```bash
# Check for German umlauts
grep -r "[äöüÄÖÜß]" export/

# Verify file encoding (should show UTF-8)
file export/pages/*/body.html
```

### Common Issues
| Issue | Cause | Solution |
|-------|-------|----------|
| "Malformed UTF-8" error | Curl encoding issue | Script uses proper encoding |
| Characters appear as `?` on site | WordPress charset wrong | Check wp-config.php DB_CHARSET |
| JSON parse errors | Corrupted export files | Re-export from source |

### When Editing Files Before Import
- Use the Edit tool which preserves UTF-8 encoding
- Never use sed or awk that might corrupt encoding
- Validate JSON files after manual edits
