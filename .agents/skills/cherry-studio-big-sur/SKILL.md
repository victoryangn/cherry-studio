---
name: cherry-studio-big-sur
description: Maintain Cherry Studio's macOS Big Sur (11.x) compatibility. Use when syncing upstream updates, building new versions, or troubleshooting Electron compatibility issues on older macOS versions.
---

# Cherry Studio macOS Big Sur Compatibility

This skill helps maintain Cherry Studio's compatibility with macOS Big Sur (11.x).

## Background

Electron 38+ dropped support for macOS Big Sur due to dependency on `QuickLookUI.framework` which is not available on macOS 11.x. This fork uses Electron 37.x to maintain compatibility.

## Quick Start

```bash
# 1. 检查是否有新版本
git fetch origin
LOCAL=$(grep '"version"' package.json | head -1 | sed 's/.*"\([0-9.]*\)".*/\1/')
REMOTE=$(git show origin/main:package.json | grep '"version"' | head -1 | sed 's/.*"\([0-9.]*\)".*/\1/')
echo "本地版本: $LOCAL"
echo "远程版本: $REMOTE"

if [ "$LOCAL" = "$REMOTE" ]; then
  echo "✓ 无新版本，无需编译"
else
  echo "⚠ 发现新版本 $REMOTE，开始编译..."
  # 继续执行下面的编译流程
fi
```

## Workflow: Sync & Build (仅当有新版本)

```bash
# === Step 1: 检查版本 ===
git fetch origin
LOCAL=$(grep '"version"' package.json | head -1 | sed 's/.*"\([0-9.]*\)".*/\1/')
REMOTE=$(git show origin/main:package.json | grep '"version"' | head -1 | sed 's/.*"\([0-9.]*\)".*/\1/')

if [ "$LOCAL" = "$REMOTE" ]; then
  echo "✓ 当前已是最新版本 $LOCAL，无需操作"
  exit 0
fi

echo "发现新版本: $LOCAL → $REMOTE"

# === Step 2: 拉取更新 ===
git merge origin/main --no-edit

# === Step 3: 恢复 Big Sur 兼容性 ===
pnpm add electron@^37.0.0 -D

# === Step 4: 安装依赖 ===
pnpm install

# === Step 5: 构建 ===
pnpm build

# === Step 6: 打包 ZIP ===
npx electron-builder --mac --x64 --config.mac.target=zip

# === Step 7: 测试 ===
pnpm dev

# === Step 8: 提交到 Fork ===
git add package.json pnpm-lock.yaml
git commit -m "fix: downgrade Electron to 37.x for macOS Big Sur compatibility"
git push fork main

echo "✓ 编译完成: dist/Cherry-Studio-$REMOTE-x64.zip"
```

## Build Output

编译成功后，安装包位于：
```
dist/Cherry-Studio-{version}-x64.zip
```

解压后运行 `Cherry Studio.app` 即可使用。

## Build Commands Reference

| Command | Description |
|---------|-------------|
| `pnpm build` | Full build (typecheck + electron-vite) |
| `pnpm build:check` | Lint + test (before commit) |
| `npx electron-builder --mac --x64 --config.mac.target=zip` | Build ZIP only (推荐) |

## Troubleshooting

### QuickLookUI Error

```
dyld: Library not loaded: /System/Library/Frameworks/QuickLookUI.framework/...
Reason: image not found
```

**Solution:** Electron 版本太新，降级：
```bash
pnpm add electron@^37.0.0 -D
```

### DMG Build Fails

```
dyld: Library not loaded: /usr/local/opt/gettext/lib/libintl.8.dylib
```

**Solution:** DMG builder 在 Big Sur 上有兼容性问题，使用 ZIP 格式：
```bash
npx electron-builder --mac --x64 --config.mac.target=zip
```

### Build Fails After Sync

```bash
rm -rf node_modules out dist
pnpm install
pnpm build
```

### Node.js Version Warnings

- 项目要求 Node.js >= 24.11.1
- macOS Big Sur 使用 Node.js 22.x
- 警告正常，功能不受影响

## Native Modules Compatibility

所有原生模块在 macOS Big Sur 上测试通过：
- `@napi-rs/system-ocr` - OCR ✓
- `sharp` - 图像处理 ✓
- `@libsql/client` - 数据库 ✓
- `selection-hook` - 文本选择 ✓
- `@napi-rs/canvas` - Canvas ✓

## Git Remotes

| Remote | URL | Purpose |
|--------|-----|---------|
| `origin` | CherryHQ/cherry-studio | 上游 (官方) |
| `fork` | victoryangn/cherry-studio | 你的 Big Sur fork |

## Code Signing

个人构建会显示代码签名警告，这是正常的。首次运行时：右键点击应用 → 打开 → 点击"打开"。
