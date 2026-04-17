# Hand Gesture HCI System

Control your computer cursor and slide presentations using only your hand — no mouse, no clicker.

## Features

| Gesture | Action |
|---|---|
| Move index finger | Move mouse cursor |
| Pinch (thumb ↔ index) | Left click |
| Swipe right | Next slide (→) |
| Swipe left | Previous slide (←) |
| Fist | Reserved / scroll mode |

---

## Project Structure

```
├── main.py               # Entry point — orchestrates the full pipeline
├── hand_tracker.py       # MediaPipe Hands wrapper (21 keypoints)
├── gesture_engine.py     # EMA smoothing + pinch / swipe / fist detection
├── screen_controller.py  # PyAutoGUI cursor, click, slide navigation
├── hud_renderer.py       # OpenCV overlay (cursor, gesture label, FPS)
├── config.py             # All tunable constants in one place
└── requirements.txt
```

---

## Setup

### 1. Prerequisites

- Python 3.9 – 3.11 recommended
- A working webcam

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. macOS — Camera & Accessibility permissions

On macOS you must grant two permissions to your terminal / IDE:

1. **Camera**: System Preferences → Privacy & Security → Camera → enable Terminal (or your IDE)
2. **Accessibility**: System Preferences → Privacy & Security → Accessibility → enable Terminal

PyAutoGUI requires Accessibility access to control the mouse.

### 4. Run

```bash
python main.py
```

Press **Q** inside the preview window to quit cleanly.

---

## Tuning

All thresholds are in `config.py` — no magic numbers elsewhere.

| Constant | Default | Effect |
|---|---|---|
| `EMA_ALPHA` | 0.30 | Cursor smoothness (lower = smoother, laggier) |
| `PINCH_THRESHOLD` | 0.055 | How close thumb & index must be for a click |
| `PINCH_DEBOUNCE_SEC` | 0.60 | Minimum time between clicks |
| `SWIPE_MIN_X_DELTA` | 0.16 | How far the swipe must travel |
| `SWIPE_HISTORY_LEN` | 14 | Frames observed for swipe detection |
| `CURSOR_MARGIN_X/Y` | 0.10 | Edge inset for full-screen cursor reach |

---

## How It Works

```
Webcam
  │
  ▼
HandTracker (MediaPipe Hands — 21 keypoints)
  │  Landmark 8 = Index tip   → cursor position
  │  Landmark 4 = Thumb tip   → pinch detection
  │
  ▼
GestureEngine
  ├─ EMAFilter2D            → smooth cursor (removes jitter)
  ├─ PinchDetector          → Euclidean dist(KP4, KP8) < threshold
  ├─ SwipeDetector          → deque of KP8 x-positions over N frames
  └─ FistDetector           → count of closed fingers ≥ 4
  │
  ▼
ScreenController (PyAutoGUI)
  ├─ move_cursor(x, y)
  ├─ click(x, y)
  ├─ next_slide()  → pyautogui.press("right")
  └─ previous_slide() → pyautogui.press("left")
  │
  ▼
HUDRenderer (OpenCV overlay)
  ├─ Cursor circle (colour = gesture state)
  ├─ Gesture label panel
  ├─ Pinch proximity bar
  ├─ Active zone rectangle
  └─ FPS counter
```
