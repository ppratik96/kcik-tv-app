# KickTV — Google TV Client with VOD & Live DVR Chat Overlay

[![Build and Release KickTV](https://github.com/ppratik96/kcik-tv-app/actions/workflows/kicktv-release.yml/badge.svg)](https://github.com/ppratik96/kcik-tv-app/actions/workflows/kicktv-release.yml)
[![Latest Release](https://img.shields.io/github/v/release/ppratik96/kcik-tv-app?color=53FC18&label=KickTV%20APK)](https://github.com/ppratik96/kcik-tv-app/releases/latest)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

**KickTV** (`tel.pratikpa.kicktv`) is an unofficial **Google TV / Android TV** (and Android mobile) client for Kick.com built around watching **full-screen past broadcasts (VODs)** and **live streams with DVR rewind**, complete with a **playhead-synchronized translucent chat overlay** and modern Smart-TV remote controls.

---

## 🔀 Upstream Fork & Attribution

This project is a fork of **[xacnio/kcik-tv-app](https://github.com/xacnio/kcik-tv-app)** (**KCIKTV** by **Alperen Cetin / [@xacnio](https://github.com/xacnio)**, also evolved from earlier `SmartTVKick` Android TV efforts).

Upstream **KCIKTV** provides the rock-solid Kotlin foundation—native **Amazon IVS 1080p60** low-latency playback, QR-code TV authentication, Centrifugo WebSocket live chat, animated Kick emotes/badges, and mobile support—while upstream's Android TV UI only supported live channel zapping with a side-panel chat that shrank the video.

**What this fork (`ppratik96/kcik-tv-app`) adds and changes:**

1. **Native TV VOD Playback + 250ms Synchronized Chat Replay (`TvVodManager`)**
   - Browse and play any streamer's past broadcasts directly on Google TV (newest VODs sorted chronologically first).
   - Zero-drift chat alignment using variant HLS `#EXT-X-PROGRAM-DATE-TIME` tags (with automatic fallback to `livestream.start_time` and UUIDv7 embedded millisecond timestamps).
   - Continuous 60-second lookahead buffering in verified 5-second windows so quiet chat stretches never stall replay, plus instant microsecond-cursor chat backfill whenever you scrub forward or backward.
   - Automatic watch-progress persistence (`Resume` where you left off or `Start from 00:00`).
2. **Fullscreen Translucent Floating Chat Overlay**
   - Renders chat directly on top of fullscreen 16:9 video (`0%` pure floating drop-shadowed text by default, or `20%–75%` theatre scrims on either the **Left** or **Right** edge) without shrinking the video frame.
   - Keeps Live WebSocket and VOD/DVR chat continuously buffered in the background when toggled off (`Chat: Off` -> `Chat: On` is instant and never drops live updates).
   - Optional VOD bot filtering (`BotRix`, `KickBot`, `Nightbot`, `StreamElements`, etc.) and relative `MM:SS` / `H:MM:SS` VOD timestamps.
3. **Unified Smart-TV HUD & Live DVR Rewind**
   - Redesigned bottom gradient player HUD with vivid red timeline seekbar, status badge (`● LIVE`, `● DVR`, `▶ VOD`, `⏸ PAUSED`), and Kick neon-green (`#53FC18`) focused pill controls (`-10s`, `Pause`/`Play`, `+10s`, `Chat: On/Off`, `Go Live`, `VODs`, `1.0x–2.0x Speed`, `Quality`, `Chat Overlay`).
   - Pressing **Left (`◀`)** or pausing on a live broadcast seamlessly transitions into Kick's growing HLS DVR stream and replays historical chat in lockstep with the rewound broadcast until jumping back to **`Go Live`**.
4. **Following-First Startup & Offline Streamer VOD Access**
   - Automatically opens your **Following** list on app launch when signed in.
   - Offline followed channels remain selectable in the sidebar (`Offline • Select for VODs`) to jump straight into their VOD archive.
5. **Independent Package ID & Automated OTA Releases**
   - Packaged as `tel.pratikpa.kicktv` so it can be installed alongside the upstream app without signature or package conflicts.
   - GitHub Actions (`kicktv-release.yml`) automatically builds, signs, verifies, and publishes `KickTV.apk` on every push to `master`, powering built-in OTA updates inside the TV app.

---

## 📺 Installing on Google TV / Android TV

1. Install **Downloader** from the Google Play Store on your Google TV / Android TV.
2. Enter **`pratikpa.tel/tv`** (or download [`KickTV.apk`](https://github.com/ppratik96/kcik-tv-app/releases/latest/download/KickTV.apk) directly from [Releases](https://github.com/ppratik96/kcik-tv-app/releases/latest)) and install **KickTV**.
3. Future updates can be installed directly from **KickTV** via the in-app OTA update prompt (or **Settings → Update Channel**).

---

## 🎮 Google TV Remote Controls

| Remote Button | Action (Live & VOD) |
| :--- | :--- |
| **Center (`OK`)** | **Play / Pause** (reveals timeline HUD; commits any active scrub) |
| **Left / Right (`◀` / `▶`)** | **Live Rewind / Fast-Forward** `-10s` / `+10s` (hold to accelerate `±30s` → `±1m` → `±2m`) with synced chat |
| **Down (`▼`)** | Focus bottom **Pill Controls** (`-10s`, `Pause`, `+10s`, `Chat: On/Off`, `Go Live`, `VODs`, `Speed`, `Quality`, `Chat Overlay`) |
| **Up (`▲`)** | Show / hide timeline HUD (or move focus from bottom pills back to timeline) |
| **Back (`←`)** | Open **Following / Channels** sidebar (press `◀` from the sidebar to open the Left Drawer for Search, VODs & Settings) |

---

## 🏗️ Project Structure

```text
app/src/main/java/dev/xacnio/kciktv/
├── tv/                         # Google TV / Android TV (D-Pad) Implementation
│   ├── PlayerActivity.kt       # Unified Smart-TV HUD, Live DVR, Floating Chat & Sidebars
│   └── TvVodManager.kt         # TV VOD Playback, Scrubbing & 250ms Chat Replay Engine
│
├── mobile/                     # Mobile / Tablet (Touch) Implementation (from upstream)
│   ├── ui/                     # Mobile UI Managers
│   ├── LoginActivity.kt        # Mobile Authentication UI
│   └── MobilePlayerActivity.kt # Main Mobile Activity
│
└── shared/                     # Shared Core & Data Layer
    ├── data/
    │   ├── api/                # Retrofit Kick API Services
    │   ├── chat/               # Centrifugo WebSocket Chat Client
    │   ├── model/              # API & Domain Models
    │   ├── prefs/              # SharedPreferences (Auth, Overlay & VOD Watch Progress)
    │   └── repository/         # Channel, VOD History, Auth & GitHub OTA Repositories
    ├── ui/adapter/             # ChatAdapter (Drop-Shadow Overlay + Emotes) & Channel Adapters
    ├── util/                   # DateParseUtils (ISO + UUIDv7 ms), QRUtils, Emote Helpers
    ├── LauncherActivity.kt     # Device Mode Router (TV vs Mobile)
    └── PlaybackService.kt      # MediaSession & Background Playback Service
```

---

## 🔧 Building from Source

### Prerequisites
- JDK 17
- Android SDK 34 (`minSdk 23`, `targetSdk 34`)
- Gradle 8.5+

### Local Build
```bash
git clone https://github.com/ppratik96/kcik-tv-app.git
cd kcik-tv-app

# Copy mock Firebase config if you don't use custom Firebase analytics
cp app/google-services.json.example app/google-services.json

# Verify locale string table integrity (enforced in CI)
python3 scripts/verify_translations_strict.py

# Build Debug or Signed Release APK
./gradlew assembleDebug
./gradlew assembleRelease
```

---

## ⚠️ Disclaimer

Unofficial, independent project — **not affiliated with, authorized by, or endorsed by Kick.com, Blerp, or any other service it connects to**. All trademarks belong to their respective owners and are used for identification only. It hosts no content of its own and connects only to publicly reachable endpoints.

## 📝 License & Credits

- Distributed under the [MIT License](LICENSE).
- Original **KCIKTV** application and core architecture by **[xacnio (Alperen Cetin)](https://github.com/xacnio/kcik-tv-app)**.
- Google TV VOD chat replay engine, unified Smart-TV HUD, Live DVR TV integration, and translucent chat overlay fork maintained by **[Pratik Patel (@ppratik96)](https://github.com/ppratik96)**.
