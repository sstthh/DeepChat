# 深聊 · DeepChat

English | [简体中文](README.md)

An **Android client** built on top of [DeepSeek's official chat-demo](https://github.com/deepseek-ai/chat-demo). It wraps that chat frontend in a minimal native shell and adds the things you need for daily use.

> **This codebase is AI-generated.** The Android shell, the frontend modifications and this document were produced by an AI coding agent under human direction; a human set the requirements, reviewed the result and published it. The code builds and runs, but evaluate it yourself before relying on it.
>
> **Unofficial project.** Not affiliated with, endorsed by, or authorized by DeepSeek in any way. See [Disclaimer](#disclaimer).

---

## What this is

The official chat-demo is a purely static page: paste your own API key and the browser talks to `api.deepseek.com` directly. It works well on a desktop browser, but on a phone the experience falls apart — no icon, address bar, state lost when backgrounded, images do nothing when tapped.

This project does two things:

1. **A minimal WebView shell** (`android/`) that bundles the frontend into an APK and provides the app icon, fullscreen, back handling, camera, microphone and file bridges.
2. **A set of frontend changes** (`frontend/`) covering what the official demo never implemented.

The shell has exactly one third-party dependency: `androidx.webkit` (used to serve local assets over an https origin, without which IndexedDB is unreliable).

## Features

**Added on top of the official chat-demo**

| Feature | Notes |
|---|---|
| Per-chat role prompt | Set per conversation, pre-filled when creating a new one. The server overrides the `system` field with its own prompt, so the setting is also injected as a leading Q&A pair to actually take effect |
| Token usage per turn | `input · output · context%` under each reply, single line, ellipsized on narrow screens |
| Remote file manager | Lists files uploaded to your account (`GET /anthropic/v1/files`) and deletes them individually |
| Local attachment manager | Lists attachment copies on the device, shows which chats reference them, deletes or prunes unreferenced ones |
| Photo capture | Opens the system camera (full camera app), compresses and attaches the shot |
| Image viewer | Fullscreen view with pinch-to-zoom, pan and double-tap |
| Import/export progress | Visible progress for long tasks; an indeterminate animation while the total is unknown |
| Reasoning effort | Low / Standard / Max in settings, sent as `output_config.effort` (Anthropic format) |
| Motion | Transitions for messages, dialogs, menus and the drawer; respects `prefers-reduced-motion` |
| Clear cache | Removes staged import files and the WebView cache |

**What the shell provides**

- Edge-to-edge rendering: transparent status/navigation bars, page background fills the screen
- Layered back handling: dismiss keyboard → close dialog → close drawer → back to new chat → double-press to exit
- Microphone permission bridge (WebView needs its own `onPermissionRequest` implementation)
- Large ZIP imports are copied to app storage first, sidestepping Android SAF random-read performance
- Fallbacks: microphone constraint downgrade, audio sample-rate resampling

## Repository layout

```
shenliao/
├─ android/                  Android shell (Kotlin, Gradle)
│  ├─ app/src/main/
│  │  ├─ java/.../MainActivity.kt
│  │  ├─ assets/www/         frontend build output (placeholder for now)
│  │  └─ res/
│  ├─ prepare-assets.ps1     copies the frontend dist into assets
│  └─ build-apk.ps1          locates JDK/SDK/Gradle and builds
├─ frontend/                 modified chat-demo source (TypeScript + React)
│  └─ src/
├─ LICENSE                   MIT (from upstream)
└─ NOTICE.md                 attribution and change log
```

## Building

**Requirements:** Node.js 22.12+, pnpm, JDK 17+, Android SDK (`platforms;android-35`, `build-tools;35.0.0`).

```powershell
# 1. Build the frontend
cd frontend
pnpm install
pnpm build            # output in frontend/dist

# 2. Copy it into the shell
cd ..\android
.\prepare-assets.ps1 -Dist ..\frontend\dist

# 3. Build the APK
.\build-apk.ps1
# android\app\build\outputs\apk\debug\app-debug.apk
```

`build-apk.ps1` auto-detects the JDK (`JAVA_HOME`, then Android Studio's bundled JBR), the Android SDK and Gradle. Pass `-Jdk` / `-Sdk` / `-Gradle` to override.

You can also open `android/` in Android Studio and build from there.

### Two run modes

| | Bundled (default) | Remote |
|---|---|---|
| Page source | `assets/www` inside the APK | a URL passed as `-PchatDemoUrl=<url>` |
| Offline | UI still opens | blank screen |
| Updates | re-package | follows the remote automatically |

```powershell
.\build-apk.ps1 -Remote https://cdn.deepseek.com/chat-demo/
```

## Usage

1. Install the APK, open Settings and paste your own DeepSeek API key.
2. Start chatting.

Everything stays on the device: conversations and attachment blobs live in IndexedDB, settings and the key in localStorage. **There is no server** — no request leaves the app except to `api.deepseek.com`.

## Known limitations

- **Not an official app**; the API may change at any time
- Attachments exist both locally and on DeepSeek's servers. Server-side copies **cannot be downloaded**, so deleting the local copy means that image is gone for good
- Custom role prompts are not included in exported history ZIPs (the archive format is the official v3, unextended)
- The context window is assumed to be 1M (`CONTEXT_WINDOW` in `frontend/src/utils/format.ts`); change it if your model differs
- Only verified on Android; `minSdk 29` / `targetSdk 34`

## Disclaimer

This is a personal, educational third-party client. It is **not affiliated with DeepSeek** and has not been authorized, endorsed or supported by them. The repository contains no official trademarks or brand assets; the app icon is an original vector drawing.

You need your own DeepSeek API key, and you pay for your own usage. Please comply with the [DeepSeek Terms of Use](https://cdn.deepseek.com/policies/en-US/deepseek-terms-of-use.html).

Provided **as is**, without warranty of any kind. The code is AI-generated and may contain undiscovered defects; use it with an understanding of what it does, at your own risk.

## License

MIT — see [LICENSE](LICENSE). The frontend is derived from [deepseek-ai/chat-demo](https://github.com/deepseek-ai/chat-demo) (MIT); see [NOTICE.md](NOTICE.md) for attribution and the list of changes.
