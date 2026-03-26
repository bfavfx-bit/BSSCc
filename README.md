# BSSC — Badminton Smash Speed Collection
### Native Android App (Kotlin · CameraX · Room · MVVM)

---

## 📱 App Overview

BSSC is a native Android app that records or imports badminton smash videos and calculates the shuttlecock speed using frame-by-frame motion analysis.

### Screens
| Screen | Description |
|---|---|
| ⚡ **Record** | Live camera recording OR pick a video from gallery |
| 📋 **History** | All saved sessions with best/avg stats and swipe-to-delete |
| 🏆 **Leaderboard** | Podium + ranked list of players by best smash speed |

### Speed Grades
| Grade | Speed |
|---|---|
| 🔴 ELITE | 320+ km/h |
| 🔵 PRO | 250–319 km/h |
| 🟢 GOOD | 200–249 km/h |
| ⚫ AVERAGE | Below 200 km/h |

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Language | Kotlin |
| UI | XML Layouts + ViewBinding |
| Camera | CameraX (camera-video, camera-view) |
| Database | Room (SQLite) |
| Architecture | MVVM + LiveData + Repository |
| Navigation | Jetpack Navigation Component |
| DI | Manual (ViewModel + Repository) |
| Min SDK | 26 (Android 8.0) |
| Target SDK | 34 (Android 14) |

---

## 🚀 Setup Instructions

### Prerequisites
- **Android Studio Hedgehog** (2023.1.1) or newer
- JDK 17
- Android device / emulator running API 26+

### Step 1 — Open in Android Studio
```
File → Open → Select the BSSC/ folder
```

### Step 2 — Add the Font (Required)
1. Visit https://fonts.google.com/specimen/Bebas+Neue
2. Download and extract the zip
3. Copy `BebasNeue-Regular.ttf` to:
   ```
   app/src/main/res/font/bebas_neue.ttf
   ```

### Step 3 — Add jitpack repository (for MPAndroidChart)
In your **root** `build.gradle`, add inside `allprojects > repositories`:
```groovy
maven { url 'https://jitpack.io' }
```

### Step 4 — Sync & Build
```
File → Sync Project with Gradle Files
Build → Make Project
```

### Step 5 — Run
Connect an Android device with USB Debugging enabled (or use an emulator), then:
```
Run → Run 'app'
```

---

## 📂 Project Structure

```
BSSC/
├── app/src/main/
│   ├── java/com/bssc/app/
│   │   ├── data/
│   │   │   ├── SmashSession.kt       # Room entity
│   │   │   ├── SmashSessionDao.kt    # DAO queries
│   │   │   ├── BsscDatabase.kt       # Room database singleton
│   │   │   └── SmashRepository.kt    # Data repository
│   │   ├── ui/
│   │   │   ├── MainActivity.kt       # Nav host + bottom nav
│   │   │   ├── MainViewModel.kt      # Shared ViewModel
│   │   │   ├── record/
│   │   │   │   ├── RecordFragment.kt # Camera + gallery + analysis
│   │   │   │   └── ResultActivity.kt # Speed result display
│   │   │   ├── history/
│   │   │   │   ├── HistoryFragment.kt
│   │   │   │   └── SessionAdapter.kt
│   │   │   └── leaderboard/
│   │   │       ├── LeaderboardFragment.kt
│   │   │       └── LeaderboardAdapter.kt
│   │   └── utils/
│   │       ├── SpeedAnalyzer.kt      # Motion detection engine
│   │       └── Extensions.kt
│   └── res/
│       ├── layout/                   # All XML layouts
│       ├── drawable/                 # Shapes, icons, badges
│       ├── navigation/nav_graph.xml
│       ├── menu/bottom_nav_menu.xml
│       ├── values/colors.xml
│       ├── values/strings.xml
│       ├── values/themes.xml
│       └── font/                     # Place bebas_neue.ttf here
└── build.gradle
```

---

## ⚙️ How Speed Analysis Works

`SpeedAnalyzer.kt` pipeline:

1. **Frame extraction** — pulls up to 30 frames using `MediaMetadataRetriever`
2. **Brightest-object detection** — scans each frame for the brightest pixel cluster (shuttlecock appears bright against dark court backgrounds)
3. **Block-matching motion** — for each consecutive frame pair, searches a ±12 px radius around the detected object to find maximum displacement
4. **Pixel → metres** — uses a court-width reference (6.1 m standard doubles court = 85% of frame width) to convert pixel displacement to real-world distance
5. **Speed calculation** — `speed (m/s) = distance / time_per_frame`, converted to km/h
6. **Sanity clamp** — result is clamped to the realistic range 100–493 km/h

### Upgrade Path (Production)
Replace the heuristic in `SpeedAnalyzer.kt` with a TFLite model:
- Train a YOLO-nano model on badminton footage to detect shuttlecock bounding boxes
- Feed bounding box centroids into the existing distance → speed formula
- Achieves ±5% accuracy vs radar gun measurements

---

## 🔒 Permissions Used

| Permission | Reason |
|---|---|
| `CAMERA` | Live camera recording |
| `RECORD_AUDIO` | Audio during video recording |
| `READ_MEDIA_VIDEO` | Gallery video picker (API 33+) |
| `READ_EXTERNAL_STORAGE` | Gallery video picker (API ≤ 32) |
| `WRITE_EXTERNAL_STORAGE` | Save recorded video (API ≤ 28) |

---

## 🐛 Known Limitations

- Speed analysis accuracy improves with good lighting and a side-on camera angle
- TFLite shuttlecock model is not included (heuristic used instead)
- Recorded videos are saved to `Movies/BSSC/` on device storage

---

## 📄 License
MIT — Free for personal and commercial use.
