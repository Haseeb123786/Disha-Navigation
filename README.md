# 🧭 DISHA — Indoor Navigation for the Visually Impaired

**DISHA** (Digital Indoor Spatial Helper for Accessibility) is a real-time, AI-powered mobile app that helps blind and visually impaired users navigate indoor environments safely. It uses on-device object detection via YOLOv8 and spatial audio feedback to announce obstacles, distances, and contextual cues — all without an internet connection.

---

🏆 **Achievement:** Awarded **2nd Place** at **Mini Hackathon 2.0** hosted by ITS Engineering College for developing **DISHA**, an AI-powered offline navigation assistant for visually impaired users.

## ✨ Key Features

| Feature | Description |
|---------|-------------|
| 🎯 **Real-time Object Detection** | YOLOv8n running on-device via TensorFlow Lite at ~5–10 FPS |
| 🔊 **Spatial Audio Feedback** | Three-zone alert system: critical beeps, warning speech, ambient blips |
| 📏 **Distance Estimation** | Geometric focal-length heuristic estimates obstacle distance without LiDAR |
| 🧠 **5-Stage Reasoning Engine** | Filters false positives, tracks persistence, classifies threat zones |
| 📵 **Fully Offline** | No internet or cloud API required — everything runs on the phone |
| 🔒 **Privacy-First** | Camera frames are processed locally and never stored or transmitted |

---

## 🏗️ Architecture

```
Camera (JPEG stream)
    │
    ▼
┌─────────────────────┐
│   Flutter Isolate    │  ← Background CPU thread
│   JPEG → Decode →   │
│   Rotate → Resize → │
│   RGB Tensor [1,3,   │
│   320,320]           │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│   YOLOv8n TFLite    │  ← 4-thread inference
│   80 COCO classes   │
│   NMS filtering     │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Reasoning Engine   │  ← 5-stage pipeline
│  1. Size filter     │
│  2. Class filter    │
│  3. Ground filter   │
│  4. Path filter     │
│  5. Temporal tracker │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│   Audio Service     │
│  • TTS speech       │
│  • Critical beeps   │
│  • Ambient blips    │
└─────────────────────┘
```

---

## 📁 Project Structure

```
lib/
├── main.dart                     # App entry point, Provider setup
├── controllers/
│   └── disha_controller.dart     # Pipeline orchestrator (Camera → YOLO → Audio)
├── core/
│   ├── constants.dart            # Tunable thresholds, zones, model config
│   └── models.dart               # Detection, BoundingBox, ConfirmedObstacle
├── services/
│   ├── camera_service.dart       # Camera init, JPEG frame streaming
│   ├── yolo_service.dart         # TFLite interpreter, Isolate tensor builder
│   ├── reasoning_engine.dart     # 5-stage obstacle filtering & tracking
│   └── audio_service.dart        # TTS + AudioPlayer for alerts
└── ui/
    ├── home_screen.dart          # Main camera UI with overlay
    └── detection_painter.dart    # Bounding box & label renderer

assets/
├── models/
│   └── yolov8n_320.tflite        # YOLOv8 Nano (320×320 input, ~12 MB)
├── labels/
│   └── coco_labels.txt           # 80 COCO class names
└── audio/
    ├── beep_critical.mp3         # Triple-beep earcon for critical zone
    └── blip_ambient.mp3          # Soft blip for ambient zone
```

---

## 🚀 Getting Started

### Prerequisites

- **Flutter** ≥ 3.0.0
- **Android device** running API 26+ (Android 8.0 Oreo or newer)
- USB debugging enabled on device

### Setup

```bash
# Clone the repository
git clone https://github.com/Haseeb123786/Disha-Navigation.git
cd Disha-Navigation

# Install dependencies
flutter pub get

# Run on connected device
flutter run
```

### Build APK

```bash
flutter build apk --release
```

The APK will be at `build/app/outputs/flutter-apk/app-release.apk`.

---

## ⚙️ Configuration

All thresholds are centralized in [`lib/core/constants.dart`](lib/core/constants.dart):

| Constant | Default | Description |
|----------|---------|-------------|
| `kConfidenceThreshold` | `0.30` | YOLO detection confidence cutoff |
| `kModelInputSize` | `320` | Model input resolution (must match `.tflite`) |
| `kZoneCriticalMax` | `0.8 m` | Objects closer than this trigger critical beeps |
| `kZoneWarningMax` | `2.0 m` | Objects in this range get TTS announcements |
| `kZoneAmbientMax` | `3.5 m` | Objects in this range get soft blip sounds |
| `kPathCenterFraction` | `0.80` | Fraction of frame width considered "in path" |
| `kMinConfirmedFrames` | `1` | Frames before first alert (temporal persistence) |
| `kAudioCooldownMs` | `3000` | Minimum ms between repeated alerts per class |

---

## 🔊 Audio Feedback Zones

| Zone | Distance | Alert Type | Example |
|------|----------|------------|---------|
| 🔴 **Critical** | < 0.8 m | Triple beep + urgent TTS | *"Warning! Person very close"* |
| 🟡 **Warning** | 0.8 – 2.0 m | TTS announcement | *"Chair ahead, 1.5 metres"* |
| 🟢 **Ambient** | 2.0 – 3.5 m | Soft blip sound | *(subtle audio cue)* |
| ⚪ **Ignore** | > 3.5 m | Silent | — |

Context objects (doors, stairs, elevators) get separate announcements with an 8-second cooldown.

---

## 🧠 Reasoning Engine — 5 Stages

1. **Size Filter** — Discard objects smaller than 20 cm in estimated real-world size
2. **Class Filter** — Categorize into Critical / Conditional / Context / Ignore buckets
3. **Ground Plane Filter** — Suppress floor-level false positives
4. **Path Filter** — Only alert for objects within the user's walking path
5. **Temporal Persistence** — Require consistent detection before alerting

---

## 📦 Dependencies

| Package | Purpose |
|---------|---------|
| `camera` | Hardware camera access & JPEG streaming |
| `tflite_flutter` | On-device TensorFlow Lite inference |
| `image` | JPEG decoding & image manipulation (in Isolate) |
| `flutter_tts` | Text-to-speech for obstacle announcements |
| `audioplayers` | Audio playback for beep/blip earcons |
| `permission_handler` | Runtime camera permission requests |
| `provider` | State management |

---

## 📱 Supported Devices

- **Platform:** Android only (API 26+)
- **Tested on:** OnePlus / OPPO (CPH2661), Android 16
- **Requirements:** Rear camera, speaker or headphones
- **Recommended:** Use with wired or Bluetooth earbuds for best spatial audio experience

---

## 🛡️ Privacy & Security

- **No data leaves the device** — all processing is local
- **No network permissions required** (INTERNET is only for Flutter debug mode)
- **Camera frames are processed in-memory** and never saved to disk
- **No analytics, tracking, or telemetry**

---

👥 Team

DISHA was developed collaboratively by Team OrvaneX during Mini Hackathon 2.0.

Team Members

- Syed Mohammad Haseeb
- Shagun Bhoj
- Jahnvi Dobal
- Savita K:
  GitHub: https://github.com/SavitaK8


## 📄 License

This project is developed as part of an accessibility initiative. See [LICENSE](LICENSE) for details.

---

<p align="center">
  <em>Built with ❤️ to make the world more navigable for everyone.</em>
</p>
