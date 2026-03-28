---
name: cherry-studio-big-sur
description: Maintain Cherry Studio's macOS Big Sur (11.x) compatibility. Use when syncing upstream updates or troubleshooting Electron compatibility issues on older macOS versions.
---

# macOS Big Sur Compatibility Maintenance

This skill helps maintain Cherry Studio's compatibility with macOS Big Sur (11.x).

## Background

Electron 38+ dropped support for macOS Big Sur due to dependency on `QuickLookUI.framework` which is not available on macOS 11.x. This fork uses Electron 37.x to maintain compatibility.

## Sync Upstream Updates

When pulling updates from the main Cherry Studio repository:

```bash
# 1. Fetch and merge upstream changes
git fetch origin
git merge origin/main

# 2. Re-apply Electron 37 downgrade if package.json was updated
pnpm add electron@^37.0.0 -D

# 3. Install dependencies
pnpm install

# 4. Test the application
pnpm dev

# 5. Commit and push to fork
git add package.json pnpm-lock.yaml
git commit -m "fix: downgrade Electron to 37.x for macOS Big Sur compatibility"
git push fork main
```

## Troubleshooting

### QuickLookUI Error

If you see this error:
```
dyld: Library not loaded: /System/Library/Frameworks/QuickLookUI.framework/...
Reason: image not found
```

Solution: Downgrade Electron to 37.x:
```bash
pnpm add electron@^37.0.0 -D
```

### Node.js Version

- This project requires Node.js >= 24.11.1
- macOS Big Sur supports Node.js up to 22.x
- Node.js 22.x works for development but may show warnings

### Native Modules

All native modules have been tested and work on macOS Big Sur:
- `@napi-rs/system-ocr` - OCR functionality
- `sharp` - Image processing
- `@libsql/client` - Database
- `selection-hook` - Text selection monitoring
- `@napi-rs/canvas` - Canvas rendering

## Key Files

- `package.json` - Electron version should be `^37.0.0`
- `pnpm-lock.yaml` - Locked dependencies

## Git Remotes

- `origin` → https://github.com/CherryHQ/cherry-studio.git (upstream)
- `fork` → Your personal fork for Big Sur compatibility
