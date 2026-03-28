---
name: cherry-studio-big-sur
description: Maintain Cherry Studio's macOS Big Sur (11.x) compatibility. Use when syncing upstream updates, building new versions, or troubleshooting Electron compatibility issues on older macOS versions.
---

# Cherry Studio macOS Big Sur Compatibility

This skill helps maintain Cherry Studio's compatibility with macOS Big Sur (11.x).

## Background

Electron 38+ dropped support for macOS Big Sur due to dependency on `QuickLookUI.framework` which is not available on macOS 11.x. This fork uses Electron 37.x to maintain compatibility.

## Workflow 1: Sync & Build Latest Version

Run this complete workflow to sync upstream updates and build a new release:

```bash
# === Step 1: Pull Upstream Updates ===
git fetch origin
git merge origin/main --no-edit

# === Step 2: Restore Big Sur Compatibility ===
# Downgrade Electron to 37.x (last version supporting Big Sur)
pnpm add electron@^37.0.0 -D

# === Step 3: Install Dependencies ===
pnpm install

# === Step 4: Update Version Number (if needed) ===
# Check current version in package.json and update if necessary
# sed -i '' 's/"version": "1.8.2"/"version": "1.8.4"/' package.json

# === Step 5: Build the Project ===
pnpm build

# === Step 6: Package macOS x64 ZIP ===
# Note: DMG fails on Big Sur due to dmg-builder dependency issues
# Use ZIP format instead
npx electron-builder --mac --x64 --config.mac.target=zip

# === Step 7: Test the Application ===
pnpm dev

# === Step 8: Commit and Push to Fork ===
git add package.json pnpm-lock.yaml
git commit -m "fix: downgrade Electron to 37.x for macOS Big Sur compatibility"
git push fork main
```

## Workflow 2: Quick Build (No Sync)

If you just want to rebuild without syncing:

```bash
pnpm build
npx electron-builder --mac --x64 --config.mac.target=zip
```

## Build Output

After successful build, the package will be at:
```
dist/Cherry-Studio-{version}-x64.zip
```

Extract and run `Cherry Studio.app` to use.

## Build Commands Reference

| Command | Description |
|---------|-------------|
| `pnpm build` | Full build (typecheck + electron-vite) |
| `pnpm build:check` | Lint + test (before commit) |
| `pnpm build:mac:x64` | Build macOS x64 (includes DMG, may fail) |
| `pnpm build:unpack` | Build without packaging (faster testing) |
| `npx electron-builder --mac --x64 --config.mac.target=zip` | Build ZIP only (recommended) |

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

### DMG Build Fails

```
dyld: Library not loaded: /usr/local/opt/gettext/lib/libintl.8.dylib
```

**Solution:** DMG builder has Big Sur compatibility issues. Use ZIP format instead:
```bash
npx electron-builder --mac --x64 --config.mac.target=zip
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

## Code Signing

The build will show a warning about missing code signing identity. This is normal for personal builds. The app will still work, but may show "unidentified developer" warnings when first launched.

To bypass: Right-click the app → Open → Click "Open" in the dialog.
