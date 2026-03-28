---
name: cherry-studio-big-sur
description: Maintain Cherry Studio's macOS Big Sur (11.x) compatibility. Use when syncing upstream updates or troubleshooting Electron compatibility issues on older macOS versions.
---

# Cherry Studio macOS Big Sur Compatibility

This skill helps maintain Cherry Studio's compatibility with macOS Big Sur (11.x).

## Background

Electron 38+ dropped support for macOS Big Sur due to dependency on `QuickLookUI.framework` which is not available on macOS 11.x. This fork uses Electron 37.x to maintain compatibility.

## Quick Sync Script

Run this complete workflow to sync upstream and rebuild:

```bash
# === Step 1: Pull Upstream Updates ===
git fetch origin
git merge origin/main --no-edit

# === Step 2: Restore Big Sur Compatibility ===
# Downgrade Electron to 37.x (last version supporting Big Sur)
pnpm add electron@^37.0.0 -D

# === Step 3: Install Dependencies ===
pnpm install

# === Step 4: Build the Project ===
pnpm build

# === Step 5: Test the Application ===
pnpm dev

# === Step 6: Commit and Push to Fork ===
git add package.json pnpm-lock.yaml
git commit -m "fix: downgrade Electron to 37.x for macOS Big Sur compatibility"
git push fork main
```

## Build Commands Reference

| Command | Description |
|---------|-------------|
| `pnpm build` | Full build (typecheck + electron-vite) |
| `pnpm build:check` | Lint + test (before commit) |
| `pnpm build:mac` | Build macOS app (arm64 + x64) |
| `pnpm build:mac:x64` | Build macOS x64 only |
| `pnpm build:unpack` | Build without packaging (faster testing) |

## Troubleshooting

### QuickLookUI Error

```
dyld: Library not loaded: /System/Library/Frameworks/QuickLookUI.framework/...
Reason: image not found
```

**Solution:** Electron version is too new. Downgrade:
```bash
pnpm add electron@^37.0.0 -D
```

### Build Fails After Sync

1. **Clean and rebuild:**
   ```bash
   rm -rf node_modules out dist
   pnpm install
   pnpm build
   ```

2. **If native modules fail:**
   ```bash
   pnpm rebuild
   ```

### Node.js Version Warnings

- Project requires Node.js >= 24.11.1
- macOS Big Sur works with Node.js 22.x
- Warnings are normal, functionality is not affected

## Native Modules Compatibility

All native modules tested on macOS Big Sur:
- `@napi-rs/system-ocr` - OCR functionality ✓
- `sharp` - Image processing ✓
- `@libsql/client` - Database ✓
- `selection-hook` - Text selection ✓
- `@napi-rs/canvas` - Canvas rendering ✓

## Git Remotes

| Remote | URL | Purpose |
|--------|-----|---------|
| `origin` | CherryHQ/cherry-studio | Upstream (official) |
| `fork` | victoryangn/cherry-studio | Your Big Sur fork |
